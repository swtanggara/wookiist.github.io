---
layout: post
title: "[Go/Golang] bufio.Scanner 사용 시 주의 사항"
category: [Golang]
tags: [blog, Golang, Go, bufio, String]
---
# `bufio.Scanner`

`bufio.Scanner`는 `bufio.Reader`를 대신하여 줄 바꿈 등으로 구분되는 텍스트 파일 등의 데이터를 읽는 편리한 인터페이스를 제공합니다. `bufio.Reader`와 눈에 띄는 가장 큰 차이점이라면, 입력을 받았을 때, 읽어 들인 값이 `\n` 을 포함하지 않는다는 점입니다. 이 덕분에 `strings.TrimSuffix(_, "\n")` 등의 작업을 추가로 해줄 필요가 없습니다.

`bufio.Scanner`와 `bufio.Reader`의 더 많은 비교는 다음 포스트를 참조해주세요.

[https://wookiist.dev/102](https://wookiist.tistory.com/102)

# 문제 발견

어제 PS를 하다 정상적으로 푼 문제가 `Runtime Error (Index out of range)`가 발생하였습니다. 같은 코드를 Python으로 작성하였을 땐 문제 없이 통과되었고, 그 코드를 로컬에서 실행하면, 정상 동작하는 하는 것을 확인했습니다.
문제의 원인을 찾기 위해 질문 게시판을 돌아다니던 중에 이전에도 비슷한 문제가 있었고, 그 분은 `(*reader).ReadString()`을 사용했을 때 컴파일러에 문제가 발생하는 것으로 결론이 지어져서, 저도 같은 원인인줄 알았습니다.
그리고 오늘 아침 관리자님의 댓글을 보고 굉장히 놀랐습니다.

> Go의 Scanner는 최대 32KB의 데이터만 한 번에 읽어들일 수 있다.
실제로 Go의 Scanner는 MaxScanTokenSize = 64 * 1024 의 값에 영향을 받습니다. 정확히는 64KB의 데이터를 한 번에 받을 수 있는 것으로 보입니다.

따라서 입력이 굉장히 큰 데이터를 입력받아야 하는 경우에는 `bufio.Scanner`가 아닌 `bufio.Reader`를 이용해서 입력 받는 것을 권장합니다.

# 한계

실제로 `bufio` 패키지를 설명하는 공식 문서에는 다음처럼 기술되어 있습니다.

> Scanning stops unrecoverably at EOF, the first I/O error, or a token too large to fit in the buffer. When a scan stops, the reader may have advanced arbitrarily far past the last token. Programs that need more control over error handling or **large tokens**, or must run sequential scans on a reader, should use bufio.Reader instead.

> **... 큰 토큰을 처리해야하는 프로그램은 bufio.Scanner 대신 bufio.Reader를 사용해야 한다**

이렇게 소개된 것처럼 큰 입력을 한 번에 받아야 할 때는 앞으로 bufio.Reader를 활용해야 할 것으로 보입니다.

어제 풀리지 않았던 문제도 bufio.Reader로 수정해서 제출하니 정답 처리가 되었습니다.

# 참고

- [https://golang.org/pkg/bufio/#Scanner](https://golang.org/pkg/bufio/#Scanner)
- [https://wookiist.dev/102](https://wookiist.dev/102)