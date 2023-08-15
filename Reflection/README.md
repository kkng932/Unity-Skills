# Reflection 
런타임 중에 메타 정보를 알아내는 기능  
성능 상의 문제로 주로 게임 시작 전에 엑셀, tsv 파일 등의 데이터를 Scriptable Object에 저장하는 방식으로 많이 사용하였다.  

제네릭과 같이 활용해서 코드를 재활용하는 것에 집중하였다.  
ScriptableObject를 상속하고 멤버변수, 클래스의 이름을 테이블과 같이 하면 tsv(엑셀)의 개별의 파싱 클래스를 만들지 않아도 자동으로 분류가 되는 로직이다.  

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
        var noteData = ReadFromTsv<Note>();
        for(int i=0;i<noteData.TNote.Count;i++)
        {
            noteData.TNote[i].beats = new int[16];
            for(int j=0;j<noteData.TNote[i].beats.Length;j++)
            {
                Debug.Log("j : " + j);
                var currNote = noteData.TNote[i];
                var fieldInfo = currNote.GetType().GetField("beat" + (j + 1).ToString());
                noteData.TNote[i].beats[j] = (int)fieldInfo.GetValue(currNote);
                Debug.Log(fieldInfo.GetValue(currNote));
            }
        }
        AssetDatabase.CreateAsset(noteData, "Assets/Resources/Data/Table/Note.asset");

        var musicList = ReadFromTsv<MusicList>();
        AssetDatabase.CreateAsset(musicList, "Assets/Resources/Data/Table/MusicList.asset");



        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();

    }
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
                // GameData 필드의 이름 
                string fiName = fi.Name;
                Debug.Log(fiName + " tsv 파일 읽는 중 ... ");
                string[] lines = File.ReadAllLines("Assets/Tsvs/TBabySeeOtter - " + fiName + ".tsv");


                // 데이터를 담을 리스트 인스턴스 
                var dataList = Activator.CreateInstance(fiType) as IList;

                // 어떤 타입 리스트인지 저장 
                var currType = fiType.GetGenericArguments()[0];

                // 속성 값 읽어옴 (두번째 줄부터 )

                List<string> columns = lines[1].Split('\t').ToList();


                for (int i = 2; i < lines.Length; i++)
                {
                    var currentRow = lines[i];

                    if (lines[i].StartsWith("//"))
                    {
                        //Debug.Log("주석 들어간 값 : "+lines[i]);
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
    private static object GetInstance(Type currType, List<string> columns, string currentRow)
    {
        // 각 행마다 저장시킬 인스턴스 생성 
        var temp = Activator.CreateInstance(currType);
        string[] cells = currentRow.Split('\t');

        for (int i = 0; i < columns.Count; i++)
        {
            var cell = cells[i];
            var fieldName = columns[i];
            //Debug.Log("속성 이름 : " + fieldName+", 넣을 값 : "+ cell.ToString());

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
