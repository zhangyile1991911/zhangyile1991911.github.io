---
title: ShaderGraphの練習2
author: zhangyile
date: 2023-12-11 13:02:00 +0800
categories: [Shader Graph]
tags: [Indie Game,Shader Graph]
comments: false
img_path: /assets/img/
image:
  path: shadergraph/pixel_dissolve.gif
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

>　このブログでは、ParticalSystemとShaderGraphを使用して簡単に爆発するイフェクト方法を紹介します。


### 画素化
まず普通な画像を画素化する
![Desktop View](shadergraph/pixel_step1.png)
数式
```
UV(0.5,0.5)*30 = 1.5,1.5
floor(1.5) = 1
1/30 = 0.033
```
まずUVの座標をN倍にして小数点を捨てる結果をNで割る。そうすれば、Nとして半径範囲で同じ色を使うようになります。

### ParticleSystem
ParticleSystemを使用して、物理におけるパラメータを適当に調整します。
重力を１に設定すると粒子が落ちる効果となります。
粒子の数をちょっと減少しないとframe rateが低い恐れがありますから。
![Desktop View](shadergraph/pixel_step3.png)
> 肝心な設定です


### トリック
この効果を制御したいだと思いますが、ShaderGraphでif文を使ったら、実行効率が低くなりがちですから、if文が避けた方がいいです。
if文の代わりにstepとlerp使いましょう
![Desktop View](shadergraph/pixel_step2.png)

### コード
```
public class PixelDissolve : MonoBehaviour
{
    public SpriteRenderer _spriteRenderer;

    public ParticleSystem _particleSystem;

    private bool iscomplete;
    // Start is called before the first frame update
    void Start()
    {
        iscomplete = true;
    }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space) && iscomplete)
        {
            iscomplete = false;
            var mat = _spriteRenderer.material;
            mat.SetFloat("_On",0f);
            var seq = DOTween.Sequence();
            var t1 = DOTween.To(
                () =>
                {
                    mat.SetFloat("_NumPixels",1000f);
                    return 1000f;
                }, 
                (param) =>
                {
                    mat.SetFloat("_NumPixels",param);
                    
                }, 30f, 2).SetEase(Ease.OutQuad);
           

            void T2()
            {
                _spriteRenderer.gameObject.SetActive(false);
                _particleSystem.Play();
            }

            seq.Append(t1);
            seq.AppendCallback(T2);
            seq.AppendInterval(2.0f);
            seq.OnComplete(() =>
            {
                mat.SetFloat("_On",1.5f);
                _spriteRenderer.gameObject.SetActive(true);
                iscomplete = true;
            });
        }
    }
}

```