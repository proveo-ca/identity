# Proveo's Identity

```plantuml
@startuml
<style>
  root {
    FontName Padauk;
    BackgroundColor #FAFAFA;
  }
</style>
frame "Main Colors" {
  rectangle " " as Black #181818
  rectangle " " as Gray #888888
  rectangle " " as White #FAFAFA
  rectangle " " as Main #31176B
  rectangle " " as Alt #F2C633
  rectangle " " as Accent #CB159A
}

frame "Status" {
  rectangle " " as SuccessLight #B2BD2B
  rectangle " " as SuccessDark #6D7600
  rectangle " " as ErrorLight #F5435F
  rectangle " " as ErrorDark #E20024
}

frame "Shades" {
  rectangle #888888 {
    rectangle " " as Gray1 #181818
    rectangle " " as Gray2 #585858
    rectangle " " as Gray3 #B4B3B3
    rectangle " " as Gray4 #E5E4E4
  }
    
  rectangle #31176B {
    rectangle " " as Main1 #130335
    rectangle " " as Main2 #1F0852
    rectangle " " as Main3 #452883
    rectangle " " as Main4 #5D4395
  }
    
  rectangle #F2C633 {
    rectangle " " as Alt1 #A68002
    rectangle " " as Alt2 #D2A60E
    rectangle " " as Alt3 #FFD95A
    rectangle " " as Alt4 #FFE383
  }
  
  rectangle #CB159A {
    rectangle " " as Accent1 #8A0064
    rectangle " " as Accent2 #B20081
    rectangle " " as Accent3 #CB1599
    rectangle " " as Accent4 #DE62BC
  }
}

@enduml
```


