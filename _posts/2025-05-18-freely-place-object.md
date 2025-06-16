---
title: Unityエディタの拡張--マップに自由にオブジェクトを置くツール
author: zhangyile
date: 2025-5-18 09:42:00 +0800
categories: [Work Log]
tags: [Work,Development]
comments: false
img_path: /assets/img/
image:
  path: company_without/isogashii_man.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 前提

> プランナー側がマップ配置作業を素早くさせるため、指定される範囲内でプレハブ生成ツールを作成します。

## Window概覧

![Desktop View](company_without/placeprefab01.png){: width="638" height="384" .w-75 .normal}

## プレハブを指定

![Desktop View](company_without/placeprefab02.png){: width="640" height="270" .w-75 .normal}

## ルール配置

![Desktop View](company_without/placeprefab03.png){: width="642" height="213" .w-75 .normal}

## 長方形範囲で生成する

![Desktop View](company_without/placerectangle.gif){: width="642" height="425" .w-75 .normal}


## 円範囲で生成する

![Desktop View](company_without/placecircle.gif){: width="642" height="425" .w-75 .normal}

## コード
```
using System.Collections.Generic;
using Sirenix.OdinInspector;
using Sirenix.OdinInspector.Editor;
using Sirenix.Utilities.Editor;
using UnityEngine;
using UnityEditor;
using UnityEditor.SceneManagement;

public class PlaceablePrefab
{
    [HorizontalGroup("Group", Width = 0.5f)]
    [Required]
    public GameObject Prefab;
    [HorizontalGroup("Group")]
    public int Weight = 10;
}

public class ClickToPlacePrefab : OdinEditorWindow
{
    public enum GenerateType
    {
        Circle,
        Rectangle,
    }
    
    [MenuItem("Tools/Click To Place Prefab")]
    static void Init()
    {
        GetWindow<ClickToPlacePrefab>("Click To Place Prefab");
    }
    
    [ListDrawerSettings(DraggableItems= true)]
    public List<PlaceablePrefab> prefabToPlaceWithWeight = new List<PlaceablePrefab>();

    protected override void OnImGUI()
    {
        Rect dropArea = EditorGUILayout.GetControlRect(false, 40);
        
        // Rect dropArea = GUILayoutUtility.GetRect(0, 40, GUILayout.ExpandWidth(true));
        GUI.Box(dropArea, "drop prefab to here", SirenixGUIStyles.MessageBox);
        base.OnImGUI();
        
        HandleDragAndDrop(dropArea);
    }

    private void HandleDragAndDrop(Rect dropArea)
    {
        Event evt = Event.current;
        switch (evt.type)
        {
            case EventType.DragUpdated:
            case EventType.DragPerform:
                if (!dropArea.Contains(evt.mousePosition))
                    break;
                
                DragAndDrop.visualMode = DragAndDropVisualMode.Copy;
                
                if (evt.type == EventType.DragPerform)
                {
                    DragAndDrop.AcceptDrag();
                    foreach (var obj in DragAndDrop.objectReferences)
                    {
                        if (obj is GameObject prefab && PrefabUtility.IsPartOfAnyPrefab(prefab))
                        {
                            prefabToPlaceWithWeight.Add(new PlaceablePrefab
                            {
                                Prefab = prefab,
                                Weight = 10
                            });
                        }
                    }
                }
                evt.Use();
                break;
        }
    }
    
    [Title("SceneObject")]
    [InfoBox("ParentRoot")]
    public GameObject Root;
    
    public string targetTag = "Ground";
    public float yOffset = 0f;

    public int GenerateCount = 5;
    
    public GenerateType SelectedGenerateShape = GenerateType.Circle;
    [BoxGroup("Circle Setting"),ShowIf("SelectedGenerateShape",GenerateType.Circle)]
    public float radius = 5f;

    [BoxGroup("Rectangle Setting"),ShowIf("SelectedGenerateShape",GenerateType.Rectangle)]
    public float width = 5f;
    [BoxGroup("Rectangle Setting"),ShowIf("SelectedGenerateShape",GenerateType.Rectangle)]
    public float height = 5f;

    void OnEnable()
    {
        SceneView.duringSceneGui += OnSceneGUI;
    }

    void OnDisable()
    {
        SceneView.duringSceneGui -= OnSceneGUI;
    }

    int getPrefabIndex(List<PlaceablePrefab> list,int totalWeight)
    {
        int randWeight = Random.Range(0, totalWeight+1);
        int startLine = 0;
        for (int x = 0; x < list.Count; x++)
        {
            int endLine = startLine + list[x].Weight;
            if (randWeight >= startLine && randWeight < endLine)
            {
                return x;
            }

            startLine = endLine;
        }

        return 0;
    }
    void GenerateObjectsInCircle(Vector3 center)
    {
        int totalWeight = 0;
        for (int i = 0; i < prefabToPlaceWithWeight.Count; i++)
        {
            totalWeight += prefabToPlaceWithWeight[i].Weight;
        }
        
        for (int i = 0; i < GenerateCount; i++)
        {
            Vector2 randPos = Random.insideUnitCircle;
            randPos.x += Random.Range(-radius, radius);
            randPos.y += Random.Range(-radius, radius);
            int prefabIndex = getPrefabIndex(prefabToPlaceWithWeight,totalWeight);
            Vector3 spawnPosition = center + new Vector3(randPos.x, yOffset, randPos.y);
            Quaternion rotation = prefabToPlaceWithWeight[prefabIndex].Prefab.transform.rotation;
            GameObject newObj = (GameObject)PrefabUtility.InstantiatePrefab(prefabToPlaceWithWeight[prefabIndex].Prefab,Root.transform);
            newObj.transform.position = spawnPosition;
            newObj.transform.rotation = rotation;
            
            Undo.RegisterCreatedObjectUndo(newObj, "Place " + prefabToPlaceWithWeight[prefabIndex].Prefab.name);
        }

    }

    void GenerateObjectsInRectangle(Vector3 center)
    {
        float halfWidth = width * 0.5f;
        float halfHeight = height * 0.5f;
        int totalWeight = 0;
        for (int i = 0; i < prefabToPlaceWithWeight.Count; i++)
        {
            totalWeight += prefabToPlaceWithWeight[i].Weight;
        }
        for (int i = 0; i < GenerateCount; i++)
        {
            float randX = Random.Range(-halfWidth, halfWidth);
            float randZ = Random.Range(-halfHeight, halfHeight);
            int prefabIndex = getPrefabIndex(prefabToPlaceWithWeight,totalWeight);
            // int prefabIndex = Random.Range(0, prefabToPlace.Count);
            Vector3 spawnPosition = center + new Vector3(randX, yOffset, randZ);
                    
            Quaternion rotation = prefabToPlaceWithWeight[prefabIndex].Prefab.transform.rotation;
                    
            GameObject newObj = (GameObject)PrefabUtility.InstantiatePrefab(prefabToPlaceWithWeight[prefabIndex].Prefab,Root.transform);
            newObj.transform.position = spawnPosition;
            newObj.transform.rotation = rotation;
           
            Undo.RegisterCreatedObjectUndo(newObj, "Place " + prefabToPlaceWithWeight[prefabIndex].Prefab.name);
        }
       
    }
    private void OnSceneGUI(SceneView sceneView)
    {
        if (prefabToPlaceWithWeight == null) return;
        if (!IsBeFocused) return;
        
        Event e = Event.current;
        
        if (e.type == EventType.MouseDown && e.button == 0)
        {
            Ray ray = HandleUtility.GUIPointToWorldRay(e.mousePosition);
            RaycastHit hit;
            int layerMask = 1 << LayerMask.NameToLayer("Default");
            if (Physics.Raycast(ray, out hit,Mathf.Infinity,layerMask))
            {
                if (hit.collider.CompareTag(targetTag))
                {
                    if (SelectedGenerateShape == GenerateType.Circle)
                        GenerateObjectsInCircle(hit.point);
                    else if (SelectedGenerateShape == GenerateType.Rectangle)
                        GenerateObjectsInRectangle(hit.point);
                    
                    EditorSceneManager.MarkSceneDirty(EditorSceneManager.GetActiveScene());
                }
            }
            
            e.Use();
        }
        else
        {
            Ray ray = HandleUtility.GUIPointToWorldRay(e.mousePosition);
            int layerMask = 1 << LayerMask.NameToLayer("Default");
            bool isMouseInSceneView = Physics.Raycast(ray, out RaycastHit hit,Mathf.Infinity,layerMask);
  
            Handles.color = Color.green;
            switch (SelectedGenerateShape)
            {
                case GenerateType.Circle:
                    DrawCircle(isMouseInSceneView,hit.point);
                    break;
                case GenerateType.Rectangle:
                    DrawRectangle(isMouseInSceneView,hit.point);
                    break;
            }

            if (e.type == EventType.MouseMove)
            {
                sceneView.Repaint();
            }
        }
        
    }
    
    private bool IsBeFocused => this.hasFocus;
    
    private void DrawCircle(bool isMouseInSceneView, Vector3 center)
    {
        if (!isMouseInSceneView) return;
        if (!IsBeFocused) return;
        center.y = 0;
        Handles.DrawWireDisc(center, Vector3.up, radius);
    }
    
    private void DrawRectangle(bool isMouseInSceneView,Vector3 center)
    {
        if (!isMouseInSceneView) return;
        if (!IsBeFocused) return;
    
        Vector3 halfSize = new Vector3(width / 2f, 0, height / 2f);
        
        Vector3[] corners = new Vector3[4]
        {
            center + new Vector3(-halfSize.x, 0, -halfSize.z),
            center + new Vector3(halfSize.x, 0, -halfSize.z),
            center + new Vector3(halfSize.x, 0, halfSize.z),
            center + new Vector3(-halfSize.x, 0, halfSize.z)
        };
        
        Handles.DrawPolyLine(corners[0], corners[1], corners[2], corners[3], corners[0]);
    }
}
```