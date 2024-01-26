---
title: ShaderGraphの練習4
author: zhangyile
date: 2024-01-25 14:40:00 +0800
categories: [Shader Graph]
tags: [Indie Game,Shader Graph]
comments: false
img_path: /assets/img/
image:
  path: gif/flippage.gif
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---


>　このブログでは、BaseMeshEffectとShader使用して本をめくる効果を紹介します。


## 手順
1. 頂点を追加する
2. 頂点の座標計算する
3. 正面と裏側を描く
4. URPの中で複数Pass機能を活用する

始めましょう！

## 頂点数を増やす
### 左側のイメージと右側の見た目は同じですが、頂点数が全然違います。
![Desktop View](flippage/vertex2.png)
よりスムーズにめくる効果できるようにしたいなら、頂点を追加のが必要です。
どうやって頂点を増やすか、しかも元の頂点上で増やす方法を紹介します。
![Desktop View](flippage/vertex1.png)

### A,B,C三つポイントを時計回りで繋げると一つ三角を構成するから。(Unityで三角の構成順番は時計回りでは正直、逆に裏側とされて見えなくなります。)
![Desktop View](flippage/vertex4.png)

### 三つの線の真ん中ごと一つポイント追加すると 以上の画像になりました。一つ三角が四つになりました。
![Desktop View](flippage/vertex3.png)

コード
```
public class InterpolateMesh : BaseMeshEffect
{
    //最大値は六になった理由は6500頂点を超えてはならない
    [Serialize,Range(0,6)]
    public int Level = 3;
    public override void ModifyMesh(VertexHelper vh)
    {
        var existedMesh = Unity.VisualScripting.ListPool<UIVertex>.New();
        vh.GetUIVertexStream(existedMesh);
        var gridMesh = new List<UIVertex>();
        for (int i = 0; i < existedMesh.Count; i += 3)
        {
            gridMesh.AddRange(interpolatePoint(existedMesh[i],existedMesh[i+1],existedMesh[i+2],Level));
        }

        Unity.VisualScripting.ListPool<UIVertex>.Free(existedMesh);
        vh.Clear();
        vh.AddUIVertexTriangleStream(gridMesh);
    }

    //利用递归,继续分割网格
    //再帰メソッドでメッシュに頂点を追加する
    List<UIVertex> interpolatePoint(UIVertex a,UIVertex b,UIVertex c,int depth)
    {
        if (depth == 0)
            return new List<UIVertex>(){a,b,c};
        var curList = insertMiddlePoint(a, b, c);
        List<UIVertex> aggregateList = new List<UIVertex>();
        for (int i = 0; i < curList.Count; i += 3)
            aggregateList.AddRange(interpolatePoint(curList[i], curList[i + 1], curList[i + 2], depth - 1));
        return aggregateList;
    }

    //線分の真ん中に点を増やす
    List<UIVertex> insertMiddlePoint(UIVertex a,UIVertex b,UIVertex c)
    {
        //在一个三角形中 切割出四个
        /*
         * b    c   b    bc   c
         *          ab     ca 
         * a        a
         */
        UIVertex middleOfAB = LerpUIVertex(a,b,0.5f);
        UIVertex middleOfBC = LerpUIVertex(b, c,0.5f);
        UIVertex middleOfCA = LerpUIVertex(c, a,0.5f);
        return new List<UIVertex>()
        {
            a,middleOfAB,middleOfCA,
            middleOfAB,middleOfBC,middleOfCA,
            middleOfAB,b,middleOfBC,
            middleOfCA,middleOfBC,c
        };
    }

    UIVertex LerpUIVertex(UIVertex a,UIVertex b,float percent)
    {
        UIVertex one = new UIVertex();
        one.position = Vector3.Lerp(a.position, b.position, percent);
        one.normal = Vector3.Lerp(a.normal, b.normal, percent);
        one.tangent = Vector3.Lerp(a.tangent, b.tangent, percent);
        one.color = Color32.Lerp(a.color,b.color,percent);
        one.uv0 = Vector2.Lerp(a.uv0, b.uv0,percent);
        one.uv1 = Vector2.Lerp(a.uv1, b.uv1,percent);
        one.uv2 = Vector2.Lerp(a.uv2, b.uv2,percent);
        one.uv3 = Vector2.Lerp(a.uv3, b.uv3,percent);
        return one;
    }
}
```

## 頂点の座標の変更を計算する
次はShaderCodeに遷移する
説明図１
![Desktop View](flippage/vertex5.png)
```
v2f vert (appdata v)
{
    v2f o;
    //将顶点转换到当前画布局部坐标
    //頂点をCanvasに換算する
    v.vertex = mul(_Canvas2Local,v.vertex);

    //转换后的坐标点
    float3 p = v.vertex.xyz;
    float2 dir = normalize(_Direction);

    //当前点 指向_Point 向量 在 方向上的投影距离
    //假设Direction -1,1,1
    //dotの意味は同じ方向の二つベクターなら　ゼロより大きいです。
    //しかも　結果は(_DragPoint - p.xy)というベクターはdirの上に長さです
    float dist = dot((_DragPoint - p.xy),dir.xy);
    if(dist > 0)
    {
        //沿着轴方向移动
        //軸に沿って移動する
        float2 bottom = p.xy + dist * dir.xy;
        //UNITY_PI * _Radius　周长的一半
        //UNITY_PI * _Radius　半分の外周
        float moreThanHalfCircle = (dist - UNITY_PI * _Radius);
        //当前的点 已经超过了一半的圆周长 说明不用旋转了,因为 当前坐标 已经被
        //若しくは　距離は外周の半分を超えているなら　旋転の必要がなくなりました
        //具体的理由は上の説明図１ご覧ください。
        if(moreThanHalfCircle >= 0)
        {
            //底部的坐标+直径
            float3 topPoint = float3(bottom,2*_Radius);
            p = topPoint + moreThanHalfCircle * float3(dir.xy,0);
        }
        else
        {
            float angle = UNITY_PI - dist/_Radius;
            float h = dist - sin(angle) * _Radius;
            float z = _Radius + cos(angle) * _Radius;
            float3 vD = p + h * float3(dir.xy,0);
            p = float3(vD.xy,z);
        }
        p.z = -p.z;
    }
    v.vertex.xyz = p;
    v.vertex = mul(_Local2Canvas,v.vertex);
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.uv = v.uv;
    return o;
}
```


## 正面と裏側を描く
### 裏側を描くコード
```
Pass
{
    Name "MultiPass0"
    Tags
    {
        "LightMode"="MultiPass0"
    }
    Cull Front
    Offset -1,-1 //对深度值进行偏移，用于避免Z-fighting（深度冲突）的问题。
    CGPROGRAM
    #pragma vertex vert
    #pragma fragment fragBack
    
    uniform sampler2D _BackTex;
    fixed4 fragBack(v2f i) : SV_Target
    {
       //裏側を逆にする
        float2 invertUV = float2(1.0f-i.uv.x,i.uv.y);
        float4 col = tex2D(_BackTex,invertUV);
        return col;
    }
    ENDCG
}
```


### 正面を描くコード
```
Pass
{
    Name "MultiPass1"
    Tags
    {
        "LightMode"="MultiPass1"
    }
    Cull Back
    Offset -1,-1 //对深度值进行偏移，用于避免Z-fighting（深度冲突）的问题。
    CGPROGRAM
    #pragma vertex vert
    #pragma fragment fragFront
    uniform sampler2D _FrontTex;
    fixed4 fragFront(v2f i) : SV_Target
    {
        
        float4 col = tex2D(_FrontTex,i.uv);
        return col;
    }
    ENDCG
}
```
## URPの制限
URP (Universal Render Pipeline) 環境ではSubShaderの中で一つPassしかないんです。
パスの追加にはパイプライン側の拡張も必要なので合わせて行なっていきたいと思います。
### ScriptableRenderPass継承
```
public class MultiPassFurRenderPass : ScriptableRenderPass
{
   //パスの数
    private const int MultiPassCount = 8;
    //Shaderにタグ名
    private const string LightModeTag = "MultiPass";
    private ShaderTagId[] tagIdList = null;
    // レンダリングするタイミング
    private readonly RenderPassEvent _renderPassEvent = RenderPassEvent.AfterRenderingTransparents;
    // 対象とするRenderQueue
    private readonly RenderQueueRange _renderQueueRange = RenderQueueRange.transparent;

    public MultiPassFurRenderPass()
    {
        renderPassEvent = _renderPassEvent;
        tagIdList = new ShaderTagId[MultiPassCount + 1];
        for (int i = 0; i < MultiPassCount+1; i++)
        {
            var name = LightModeTag + i.ToString();
            tagIdList[i] = new ShaderTagId(name);
        }
    }

    public override void Execute(ScriptableRenderContext context,ref RenderingData renderingData)
    {
        for (var i = 0; i < MultiPassCount + 1; i++)
        {
            var sortFlags = renderingData.cameraData.defaultOpaqueSortFlags;
            var dSettings = CreateDrawingSettings(tagIdList[i], ref renderingData, sortFlags);
            var fSetting = new FilteringSettings(_renderQueueRange, -1);
            dSettings.perObjectData = PerObjectData.None;
            context.DrawRenderers(renderingData.cullResults,ref dSettings,ref fSetting);
        }
    }
}
```
### ScriptableRendererFeature継承
```
public class MultiPassFurRendererFeature : ScriptableRendererFeature
{
    private MultiPassFurRenderPass _renderPass = null;

    public override void Create()
    {
        _renderPass = new MultiPassFurRenderPass();
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        if (_renderPass == null)
        {
            return;
        }
        //レンダリングパイプラインにパスを追加
        renderer.EnqueuePass(_renderPass);
    }
}
```
## 注意点
Levelが大きれば頂点数も倍に増やすから、実行効率が下がる恐れがある。扱いに十分気を付ける
![Desktop View](flippage/vertex6.png)
![Desktop View](flippage/vertex7.png)
![Desktop View](flippage/vertex8.png)

## プロジェクト
[**source code**](https://github.com/zhangyile1991911/FlipPage) 


## 参考資料:
https://engineering.cocone.io/2023/03/31/multipath_furshader_urp/



