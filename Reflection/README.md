# Reflection 
런타임 중에 메타 정보를 알아내는 기능  
성능 상의 문제로 게임 중에는 지양하였고 주로 게임 시작 전에 엑셀, tsv 파일 등의 데이터를 Scriptable Object에 저장하는 용도로 많이 사용하였다.  

제네릭과 같이 활용해서 코드를 재활용하는 것에 집중하였다.  
ScriptableObject를 상속하고 멤버변수, 클래스의 이름을 테이블과 같이 하면 자동으로 분류가 되는 로직이다.  



```C#
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using UnityEditor;
using UnityEngine;

public class MyMenu 
{
    [MenuItem("MyMenu/TsvToAsset")]
    public static void TsvToAsset()
    {
        var tempData = ReadFromTsv<TempDataClass>();
        AssetDatabase.CreateAsset(tempData, "Assets/Resources/Data/Table/MusicList.asset");
        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();
    }
    // Tsv 파일을 읽어 속성값에 따라 제네릭 타입을 분류, 데이터 저장 
    public static T ReadFromTsv<T>()
    {
        var result = Activator.CreateInstance<T>();
        var tType = typeof(T);
        var tFiledInfos = tType.GetFields();
        // 제네릭 필드 순회 
        foreach (var fi in tFiledInfos)
        {
            var fiType = fi.FieldType;
            // 제네릭 타입 , 리스트형인지 확인 
            if (fiType.IsGenericType && fiType.GetGenericTypeDefinition() == typeof(List<>))
            {
                string fiName = fi.Name;
                Debug.Log(fiName + " tsv 파일 읽는 중 ... ");
                string[] lines = File.ReadAllLines("경로" + fiName + ".tsv");
                
                var dataList = Activator.CreateInstance(fiType) as IList;

                // 어떤 타입 리스트인지 저장 
                var currType = fiType.GetGenericArguments()[0];

                // 속성 값 
                // 제공받은 테이블은 첫 행은 한글 이름, 두번 째 행은 영어 이름을 사용하여 두번 째 행을 활용하였다. 
                List<string> columns = lines[1].Split('\t').ToList();

                // 속성 값 읽어옴 (세번째 줄부터)
                for (int i = 2; i < lines.Length; i++)
                {
                    var currentRow = lines[i];
                    // 데이터 파일에서도 주석처리가 가능하도록 만들었다.  
                    if (lines[i].StartsWith("//"))
                    {
                        continue;
                    }
                    object temp = GetInstance(currType, columns, currentRow);
                    dataList.Add(temp);
                }
                fi.SetValue(result, dataList);
            }
        }
        return result;
    }
    // 한 행의 정보를 받아 데이터 저장 
    private static object GetInstance(Type currType, List<string> columns, string currentRow)
    {
        var temp = Activator.CreateInstance(currType);
        string[] cells = currentRow.Split('\t');

        for (int i = 0; i < columns.Count; i++)
        {
            // 현재 내용 
            var cell = cells[i];
            // 현재 속성 이름 
            var fieldName = columns[i];

            var fieldInfo = currType.GetField(fieldName);
            if (fieldInfo == null)
            {
                continue;
            }
            var ft = fieldInfo.FieldType;
            SetValue(temp, cell, fieldInfo, ft);
        }

        return temp;
    }
    // 타입 정보로 분류 
    private static void SetValue(object temp, string column, System.Reflection.FieldInfo fieldInfo, Type ft)
    {
        if (ft == typeof(int))
        {
            if (column.Equals(string.Empty))
                column = "-1";
            fieldInfo.SetValue(temp, int.Parse(column));
        }
        else if (ft == typeof(double))
        {
            if (column.Equals(string.Empty))
                column = "-1";
            fieldInfo.SetValue(temp, double.Parse(column));
        }
        else if (ft == typeof(float))
        {
            if (column.Equals(string.Empty))
                column = "-1";
            fieldInfo.SetValue(temp, float.Parse(column));
        }
        else if (ft == typeof(string))
        {
            fieldInfo.SetValue(temp, column);
        }
        else if (ft == typeof(Color))
        {
            Color tempColor = Color.clear;
            if (!column.Equals("-1"))
                ColorUtility.TryParseHtmlString("#" + column + "FF", out tempColor);
            fieldInfo.SetValue(temp, tempColor);
        }
    }
}

```

아래는 데이터를 저장할 ScriptableObject 클래스입니다. 

```C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TempDataClass : ScriptableObject
{
    public List<TTempDataClass> TTempDataClass = new List<TTempDataClass>();
}

[Serializable]
public class TTempDataClass
{
    public string tempStr;
    public int tempInt;
}
```
이름으로 데이터를 읽어 구분하므로 멤버변수의 이름을 맞춰줘야 한다.  
![image](https://github.com/kkng932/Unity-Skills/assets/88977637/1571537c-9dfe-481f-b19a-b1afe1a6e360)

참고를 위한 임시 코드    
데이터 누락에 대비하기 위한 임시 데이터를 포함한 내용이므로 사용 시에 데이터, 구조에 맞게 설정 변경이 필요하다.  
