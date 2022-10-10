# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
## СБОР, ОБРАБОТКА И ВИЗУАЛИЗАЦИЯ ТЕСТОВОГО НАБОРА ДАННЫХ.
Отчет по лабораторной работе #2 выполнил(а):
- Шубина Арина Николаевна
- РИ-210947
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
познакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity
## Постановка задачи.
В данной лабораторной работе на языке python будет реализован функционал, позволяющий генерировать стоимость товара (ресурса или игрового объекта) в виде набора данных. Созданный набор данных будет передан в google-таблицу с целью возможности дальнейшего их наглядного представляния и оптимизации. Также в этой лабораторной работе на движке Unity будет реализован функционал, позволяющий воспроизводить аудио-файлы со звуковой информацией в зависимости от значений входного набора данных из таблицы.


## Задание 1
Реализовать совместную работу и передачу данных в связке Python - Google-Sheets – Unity.

Подключила API для работы с Google Sheets:
![image](https://user-images.githubusercontent.com/114181560/194942230-7b0410b9-8bee-4fe7-890b-e6dcfd3f646f.png)

Python код для записи данных из скрипта в Google Sheets:
```py

import gspread
import numpy as np
gc = gspread.service_account(filename='unitydatascience-365116-136af73acabe.json')
sh = gc.open("UnitySheets")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon):
    i += 1
    if i== 0:
        continue
    else:
        tempInf = ((price[i-1]-price[i-2])/price[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.', ',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i-1]))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(tempInf)

```
Создала Unity проект и написал скрипт для получения данных из Google Sheets:
```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1tJI8GWYXUV2vFOgFWKrUEWLiv6cqMJVfBL5HTUi5dQY/values/Лист1?key=AIzaSyDlMi_9ukGm7qB5nvoodlSeTYTbkagnQfw");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }
    
```
Создала проект в Unity, где будут воспроизводится аудио в зависимости от полученных результатов из таблицы
![image](https://user-images.githubusercontent.com/114181560/194936954-08783e6a-1409-4647-ae0f-91f8043f4a78.png)
![image](https://user-images.githubusercontent.com/114181560/194937331-c9bb08d4-d1f6-4456-8243-f392bac08c26.png)

- Определите связанные функции. Функция модели: определяет модель линейной регрессии wx+b. Функция потерь: функция потерь среднеквадратичной ошибки. Функция оптимизации: метод градиентного спуска для нахождения частных производных w и b.


## Задание 2
### Реализовать запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1. 

```py
import numpy as np
import gspread


def model(a, b, x):
    return a * x + b


def loss_function(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    da = (1.0 / num) * ((prediction - y) * x).sum()
    db = (1.0 / num) * (prediction - y).sum()
    a = a - Lr * da
    b = b - Lr * db
    return a, b


def iterate(a, b, x, y, times):
    for i in range(times):
        a, b = optimize(a, b, x, y)
    return a, b


gc = gspread.service_account(filename='unitydatascience-365116-136af73acabe.json')
sh = gc.open("UnitySheets")
x = [3, 21, 22, 34, 54, 34, 55, 67, 89, 99]
y = [2, 22, 24, 65, 79, 82, 55, 130, 150, 199]
x, y = np.array(x), np.array(y)
n_iterations = 1
initialized = False
worksheet = sh.get_worksheet(1)


for i in range(2, 9):
    if not initialized:
        worksheet.update('A1', "Значение a")
        worksheet.update('B1', "Значение b")
        worksheet.update('C1', "Количество итераций")
        worksheet.update('D1', "Потеря (loss)")
        initialized = True

    a, b = np.random.rand(1), np.random.rand(1)
    Lr = 0.000_000_01

    a, b = iterate(a, b, x, y, n_iterations)
    prediction = model(a, b, x)
    loss = loss_function(a, b, x, y)

    a_value = a[0]
    b_value = b[0]
    worksheet.update(f'A{i}', a_value)
    worksheet.update(f'B{i}', b_value)
    worksheet.update(f'C{i}', n_iterations)
    worksheet.update(f'D{i}', loss)
    print(f"a: {a_value}, b: {b_value}, iterations: {n_iterations}, loss: {loss}")
    n_iterations *= 10


```

## Задание 3
### Самостоятельно разработать сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных в задании 2.

```py
    void Update()
    {
        if (dataSet["Mon_" + i] <= 600 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i]);
        }
        if (dataSet["Mon_" + i.ToString()] > 400 & dataSet["Mon_" + i.ToString()] < 600 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i]);
        }

         if (dataSet["Mon_" + i.ToString()] >= 400 & statusStart == false & i != dataSet.Count)
         {
        
              StartCoroutine(PlaySelectAudioGood());
              Debug.Log(dataSet["Mon_" + i]);
         }
    
    IEnumerator GoogleSheets()
    {
        var currentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1tJI8GWYXUV2vFOgFWKrUEWLiv6cqMJVfBL5HTUi5dQY/values/Лист1?key=AIzaSyDlMi_9ukGm7qB5nvoodlSeTYTbkagnQfw");
        yield return currentResp.SendWebRequest();
        var rawResp = currentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[4]));
        }
    }

```
![image](https://user-images.githubusercontent.com/114181560/194939736-737e5958-dd2a-4937-8c30-27f591130f5e.png)


## Выводы

Абзац умных слов о том, что было сделано и что было узнано.


## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
