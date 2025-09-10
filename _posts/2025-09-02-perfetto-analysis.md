---
title: Perfetto 분석하기
date: 2025-09-02 20:00:00 +0900
categories: [Linux, Android]
tags: [Android, Mobile, Rooting, kernel, Perfetto, Logging, Power]
---

# Perfetto 분석하기

## 1. Intro

Perfetto 실행시 나오는 결과 파일인 `.perfetto-trace`는 SQL DB 형식으로 저장된다. 
따라서 이 파일에서 적절한 데이터를 불러와 분석하기 위해서는 SQL을 통해서 가져와야 한다.

## 2. Perfetto UI

웹사이트 상에서 UI를 통해 쉽게 확인할 수도 있다. [Perfetto UI](https://ui.perfetto.dev/) 사이트의 우측 상단 `Trace Viewer`를 클릭해 들어가면 된다. 그리고 파일 업로드 하고 데이터 열어볼 수 있다. 계산식은 아래 글을 더 읽고 참고하길 바란다.

## 3. 코드로 분석하기

SQL로 분석하는 것도 좋지만, 더 확장성이 좋은 분석을 위해서 python을 활용해 분석하기로 하였다. 그렇지만, 똑같이 기본적인 SQL 코드는 사용한다. 전용 python module이 있어 그 module을 활용하고자 한다.

```shell
pip install perfetto
pip install pandas
```

Perfetto UI 사이트에 가보면, 여러가지 열어보다 보면 `track_id`를 확인할 수 있다. 이 `track_id`를 기준으로 우리는 여러가지 값을 가져올 수 있다. 그리고, value를 기준으로 power 값 계산이 가능하고 이 계산은 [구글 소스](https://android.googlesource.com/platform/external/perfetto/%2B/refs/heads/ui-stable/src/trace_processor/perfetto_sql/stdlib/android/power_rails.sql)를 참고했다. 

>TIP: 에너지 계산은 delta 값을 활용하면 가능하다.
{: .prompt-info}


```py
from perfetto.trace_processor import TraceProcessor
import pandas as pd

""""
[track_id]
"""
track_map = {
    # gpu
    "power.rails.gpu": 1,
    "power.rails.display": 21,

    # cpu
    "power.rails.cpu.big": 14,
    "power.rails.cpu.mid": 13,
    "power.rails.cpu.little": 15,

    # memory
    "power.rails.ddr.a": 3,
    "power.rails.ddr.b": 8,
    "power.rails.ddr.c": 2,
    "power.rails.memory.interface": 12,

    # low-dropout power domain
    "power.rails.ldo.main.a": 17,
    "power.rails.ldo.main.b": 19,
    "power.rails.ldo.sub": 7,
}

ids = list(track_map.values())
names = list(track_map.keys())

nano = 1E-9
micro = 1E-6
milli = 1E-3
tp = TraceProcessor('power_trace.perfetto-trace')

collection = pd.DataFrame()
# power collection with calculation
for i, n in zip(ids, names):
    # data access
    query = f"SELECT * FROM counter WHERE track_id={i} ORDER BY ts;"
    result = tp.query(query).as_pandas_dataframe()
    
    # data aggregation
    delta = pd.DataFrame()
    delta[f'dt_{i}'] = result['ts'].diff()
    delta[f'dt_{i}'] *= nano
    delta[f'dE_{i}'] = result['value'].diff()
    delta[f'dE_{i}'] *= micro
    delta = delta.tail(-2).reset_index(drop=True) # tailing NaN and zeros

    # power calculation
    delta[n] = delta[f'dE_{i}']/delta[f'dt_{i}']
    collection = pd.concat([collection, delta], axis=1)

# save to csv
filename = "output.csv"
collection.to_csv(filename, index=True)
```

이 코드를 활용한다면, 쉽게 분석이 가능하다.

