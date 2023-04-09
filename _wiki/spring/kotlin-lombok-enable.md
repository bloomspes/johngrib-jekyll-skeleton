---
layout  : wiki
title   : Kotlin 프로젝트에서 lombok 어노테이션 활성화가 되지 않을 때
toc     : true
date    : 2023-04-09 16:09:00 +0900
updated : 2023-04-09 16:09:00 +0900
toc     : true
public  : true
parent  : spring
comment : true
regenerate: true
---

* TOC
{:toc}

코틀린 프로젝트에서 롬복 어노테이션을 사용하는 경우 주의 할 점은, Javac compiler 주석 프로세서를 비활성화 하기 때문에, `build.gradle.kts` 파일에서 아래의 조건을 추가하면 해결 가능하다.

```Kotlin

kapt {
    keepJavacAnnotationProcessors = true
}
```

참고: https://kotlinlang.org/docs/kapt.html#keeping-java-compiler-s-annotation-processors

