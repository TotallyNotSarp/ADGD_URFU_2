# ADGD_URFU_2
Workshop #2

# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Иванов Денис Александрович
- РИ-230941
Отметка о выполнении заданий:

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

## Цель работы
Установить необходимое программное обеспечение, которое пригодится для создания интеллектуальных моделей на Python. Рассмотреть процесс установки игрового движка Unity для разработки игр.

## Задание 1
### Игровой параметр в экономической модели (Здоровье)
Ход работы:
- Выбрали в качестве изучаемого параметра Здоровье. Описали ее роль, изменения и место в экономической модели

![image](https://github.com/user-attachments/assets/837d93c0-d3b4-41ae-80c5-7af778576ab5)

![image](https://github.com/user-attachments/assets/4a18bb24-6d66-4bd6-b6d0-d496f9690778)

## Задание 2
### Визуализация изменения параметра и возможности улучшения
Ход работы:
- С помощью python и google-sheets создали таблицу с генератором значений

```python
import gspread
import numpy as np
gc = gspread.service_account(filename='unitydatascience-437209-0ed8706a4920.json')
sh = gc.open("UnityDataScience")
options = [-10, -10, 1, 2, 3, 4, 10]
hp = 30
hp_cap = 30
mon = list(range(1,20))
i = 2
sh.sheet1.update(('A' + str(i)), int(i-1))
sh.sheet1.update(('B' + str(i)), int(hp))
sh.sheet1.update(('C' + str(i)), int(hp_cap))
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        value = np.random.randint(0, 7)
        if options[value] == options[-1]:
            hp_cap += options[-1]

        if not(hp + options[value] >= hp_cap):
            hp += options[value]
            
        sh.sheet1.update(('A' + str(i)), int(i-1))
        sh.sheet1.update(('B' + str(i)), int(hp))
        sh.sheet1.update(('C' + str(i)), int(hp_cap))

        if hp <= 0:
            break
```

- Визуализировали данные
  
![image](https://github.com/user-attachments/assets/3e5e2d2e-058b-40b1-9daf-fdd4541fcf58)

- Основываясь на игровом опыте, хочу сказать, что часто возникают ситуации, когда игрока зажимает толпа врагов. В таком случае враги атакуют одновременно и способны почти мгновенно убить игрока/нанести большой урон. Возможное решение - добавить кадры неуязвимости. Также навык вампиризм на высоких уровнях способен очень сильно восполнять здоровье, что сказывается на балансе.

## Задание 3
### Отображение данных в Unity

- Настроили в Unity отображение параметра с помощью скрипта, включающего звуки и выводящего надписи в консоль

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip _goodSpeak;
    public AudioClip _normalSpeak;
    public AudioClip _badSpeak;
    private AudioSource _selectAudioSource;
    private Dictionary<string, HPInfo> _dataSet = new Dictionary<string, HPInfo>();
    private int i = 1;

    void Start()
    {
        _selectAudioSource = GetComponent<AudioSource>();
        StartCoroutine(GoogleSheets());
        InvokeRepeating("TryPlaySound", 1f, 1f);
    }

    private void TryPlaySound()
    {
        if (i == _dataSet.Count)
            return;

        int currentHP = _dataSet["Mon_" + i.ToString()].CurrentHp;
        int maxHP = _dataSet["Mon_" + i.ToString()].MaxHP;

        if (currentHP <= 0) 
            Debug.Log("DEAD");
        else if (currentHP <= maxHP / 3)
        {
            Debug.Log($"Low HP: {currentHP}/{maxHP}");
            PlaySelectAudioSound(_badSpeak);
        }
        else if (currentHP <= (maxHP / 3) * 2)
        {
            Debug.Log($"Normal HP: {currentHP}/{maxHP}");
            PlaySelectAudioSound(_normalSpeak);
        }
        else
        {
            Debug.Log($"High HP: {currentHP}/{maxHP}");
            PlaySelectAudioSound(_goodSpeak);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1CC4lHQhWVai8IR3ik3DjRkcffFUwOtfzPiiQSztpmbQ/values/List1?key=AIzaSyCLFOWoJ1PpLWuEwr2Lsq581acM7jzseBk");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            _dataSet.Add(("Mon_" + selectRow[0]), new HPInfo(int.Parse(selectRow[1]), int.Parse(selectRow[2])));
        }
    }

    private void PlaySelectAudioSound(AudioClip clip)
    {
        _selectAudioSource.clip = clip;
        _selectAudioSource.Play();
        i++;
    }
}

public struct HPInfo
{
    public HPInfo(int current, int max)
    {
        CurrentHp = current;
        MaxHP = max;
    }

    public int CurrentHp;
    public int MaxHP;
}
```

![Снимок экрана 2024-09-30 181651](https://github.com/user-attachments/assets/f7b3d8b1-0dd3-4bf2-853e-24600679db9d)

## Выводы

- Подключили Google таблицы к Unity и научились обрабатывать полученные данные.
