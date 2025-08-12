---
title: Unity 动态修改 Volume
tags: Unity
math: true
index_img: /imgs/LearnPBR/pbr.png
banner_img: /imgs/LearnPBR/pbr.png
date: 2024-05-02
typora-root-url: ../
---

如何在代码中动态修改 Unity Volume 的参数

# Unity 动态修改 Volume 参数

## 尝试直接改 Volume 参数

失败了，只有那一行有效，后面又被重写为之前设置的 0了

~~~C#
using System.Collections;
using System.Collections.Generic;
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.Playables;
using UnityEngine.Rendering;

public class EffectController : MonoBehaviour
{
    public GameObject daoguang;
    public PlayableDirector effPd;
    public VolumeStack volumeStack;
    public CutScreenVolume cutScreenVolume;
    public bool cut = false;
    // Start is called before the first frame update
    void Start()
    {
        volumeStack = VolumeManager.instance.stack;
        cutScreenVolume = volumeStack.GetComponent<CutScreenVolume>();
        if (daoguang != null)
        {
            daoguang.SetActive(true);
            effPd = daoguang.GetComponent<PlayableDirector>();
            effPd.paused += SetCutScreenState;
            effPd.stopped += SetCutScreenState;
        }
    }

    // Update is called once per frame
    void Update()
    {
        //Debug.Log(effPd.state.ToString());
        Debug.Log("EffController" + cutScreenVolume.Intensity.value);
        if(cut && effPd.state == PlayState.Paused)
        {
            //SetCutScreen();
        }
    }

    void SetCutScreenState(PlayableDirector aDirector)
    {
        this.cut = true;
        Debug.Log(aDirector.state.ToString());
        cutScreenVolume.Intensity.value = 1.0f;	// 这行有用，但是只有一次
        Debug.Log(cutScreenVolume.Intensity.value);
    }

    IEnumerator SetCutScreen()
    {
        cutScreenVolume.Intensity.value = 1.0f;
        while(cutScreenVolume.Intensity.value > 0.0f)
        {
            cutScreenVolume.Intensity.value -= Time.deltaTime * 0.5f;
            if(cutScreenVolume.Intensity.value <= 0.0f)
            {
                cut = false;
            }
            yield return null;
        }
    }
}
~~~

