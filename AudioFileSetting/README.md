## 오디오 임포트 옵션 


#### 배경음악 또는 환경음의 임포트 권장 설정법
|Load Type|Compression Format|Quality|
|------|---|---|
|Streaming|Vorbis||
|Compressed In Memory|Vorbis|70%|
  
  
#### 상황별 임포트 권장 설정법
|발생 빈도|파일의 크기|Load Type|Compression Format|
|------|---|---|---|
|빈번히 발생|작은 크기|Decompress On Load|PCM 또는 ADPCM|
|빈번히 발생|중간 크기|Compressed In Memory|ADPCM|
|가끔 발생|작은 크기|Compressed In Memory|ADPCM|
|가끔 발생|중간 크기|Compressed In Memory|Vorbis|

절대강좌! 유니티 p.277
