---
title: fuzzing101 exercise 1
description: fuzzing101 exercise 1
author: doxawahebi
date: 2026-04-03 23:40:00 +0900
categories: [pwnable]
tags: [fuzzing, fuzzing101, pwnable, xpdf, aflpluslpus]     # TAG names should always be lowercase
pin: false
math: true
mermaid: false
---


## Overview
---
Fuzzing101의 Exercise 1을 직접 실습하며, 취약점이 어떻게 발생하고 Fuzzer가 취약점을 어떻게 찾아내는지 알아보겠습니다

## CVE-2019-13288
---
```
Xpdf 4.01.01 Parser::getObj() Infinite Recursion Denial of Service Vulnerability

In Xpdf 4.01.01, the Parser::getObj() function in Parser.cc may cause infinite recursion via a crafted file. A remote attacker can leverage this for a DoS attack. This is similar to CVE-2018-16646.
```
https://www.cvedetails.com/cve/CVE-2019-13288/



### PDF Structure
---
PDF는 Object들이 얽히고설킨 거대한 데이터베이스입니다. 크게 4가지 구역(Header, Body, XRef Table, Trailer)으로 나뉩니다.

![[58e8caed9ccc003d3c9af6868857837e69cbb515-409x644.webp]]
(https://apryse.com/blog/pdf-structure-creation)

Body에서 실제 데이터는 다음과 같이 정의됩니다.
```
5 0 obj
<</Type /Catalog /Pages 2 0 R
/PageLayout/SinglePage
/PageMode/UseNone
/Page 1
/Metadata 3 0 R
>>
endobj
```

### `Parser`\:\:`getObj`
---
```cpp
Object *Parser::getObj(Object *obj, Guchar *fileKey,
		       CryptAlgorithm encAlgorithm, int keyLength,
		       int objNum, int objGen) {
  char *key;
  Stream *str;
  Object obj2;
  int num;
  DecryptStream *decrypt;
  GString *s, *s2;
  int c;

  // refill buffer after inline image data
  if (inlineImg == 2) {
    buf1.free();
    buf2.free();
    lexer->getObj(&buf1);
    lexer->getObj(&buf2);
    inlineImg = 0;
  }

  // array
  if (buf1.isCmd("[")) {
    shift();
    obj->initArray(xref);
    while (!buf1.isCmd("]") && !buf1.isEOF())
      obj->arrayAdd(getObj(&obj2, fileKey, encAlgorithm, keyLength,
			   objNum, objGen));
    if (buf1.isEOF())
      error(getPos(), "End of file inside array");
    shift();

  // dictionary or stream
  } else if (buf1.isCmd("<<")) {
    shift();
    obj->initDict(xref);
    while (!buf1.isCmd(">>") && !buf1.isEOF()) {
      if (!buf1.isName()) {
	error(getPos(), "Dictionary key must be a name object");
	shift();
      } else {
	key = copyString(buf1.getName());
	shift();
	if (buf1.isEOF() || buf1.isError()) {
	  gfree(key);
	  break;
	}
	obj->dictAdd(key, getObj(&obj2, fileKey, encAlgorithm, keyLength,
				 objNum, objGen));
      }
    }
    if (buf1.isEOF())
      error(getPos(), "End of file inside dictionary");
    // stream objects are not allowed inside content streams or
    // object streams
    if (allowStreams && buf2.isCmd("stream")) {
      if ((str = makeStream(obj, fileKey, encAlgorithm, keyLength,
			    objNum, objGen))) {
	obj->initStream(str);
      } else {
	obj->free();
	obj->initError();
      }
    } else {
      shift();
    }

  // indirect reference or integer
  } else if (buf1.isInt()) {
    num = buf1.getInt();
    shift();
    if (buf1.isInt() && buf2.isCmd("R")) {
      obj->initRef(num, buf1.getInt());
      shift();
      shift();
    } else {
      obj->initInt(num);
    }

  // string
  } else if (buf1.isString() && fileKey) {
    s = buf1.getString();
    s2 = new GString();
    obj2.initNull();
    decrypt = new DecryptStream(new MemStream(s->getCString(), 0,
					      s->getLength(), &obj2),
				fileKey, encAlgorithm, keyLength,
				objNum, objGen);
    decrypt->reset();
    while ((c = decrypt->getChar()) != EOF) {
      s2->append((char)c);
    }
    delete decrypt;
    obj->initString(s2);
    shift();

  // simple object
  } else {
    buf1.copy(obj);
    shift();
  }

  return obj;
}

```

해당 함수에서 취약점이 발생한다고 한다.

### Build
---
(gcc)
```shell
wget https://dl.xpdfreader.com/old/xpdf-3.02.tar.gz
tar -xvzf xpdf-3.02.tar.gz
cd xpdf-3.02

./configure --prefix=/home/cks/fuzzing/fuzzing_xpdf/install
make
make install
```

(afl-clang-fast)
```shell
wget https://dl.xpdfreader.com/old/xpdf-3.02.tar.gz
tar -xvzf xpdf-3.02.tar.gz
cd xpdf-3.02

export AFL_USE_ASAN=1
CC=afl-clang-fast CXX=afl-clang-fast++ ./configure --prefix=/home/cks/fuzzing/fuzzing_xpdf/install_afl_fast
make
make install
```


### Download Sample
---
1부터 생성하면 Crash file을 만들 때까지 천문학적인 비용이 들 수 있으므로 pdf를 가져온다.

```shell
mkdir pdf_examples && cd pdf_examples
wget https://github.com/mozilla/pdf.js-sample-files/raw/master/helloworld.pdf
wget http://www.africau.edu/images/default/sample.pdf
wget https://www.melbpc.org.au/wp-content/uploads/2017/10/small-example-pdf-file.pdf
```

## Fuzzing
---
```shell
afl-fuzz -m none -i ../pdf_examples -o ../out -s 123 -- $HOME/fuzzing/fuzzing_xpdf/install_afl_fast/bin/pdftotext @@ $HOME/fuzzing/fuzzing_xpdf/output
```

Error 발생 시 아래를 수행해준다.
```shell
sudo su
echo core >/proc/sys/kernel/core_pattern
exit
```



## 디버그용 컴파일
---
Crash를 발견했다면 분석을 시작하기 전에 분석을 쉽게 하기 위한한 ASAN을 적용하고 최적화가 없는 바이너리 파일을 만든다.
```shell
make clean
```

```shell
CFLAGS="-fsanitize=address -g -O0" CXXFLAGS="-fsanitize=address -g -O0 -fpermissive" LDFLAGS="-fsanitize=address" ./configure --prefix=/home/cks/fuzzing/fuzzing_xpdf/install_asan_zero
```

```shell
make
make install
```

## crash 결과 삽입
---

```shell
AddressSanitizer:DEADLYSIGNAL
=================================================================
==460331==ERROR: AddressSanitizer: stack-overflow on address 0x7ffe84860ff0 (pc 0x774ad3b26fdb bp 0x7ffe84861030 sp 0x7ffe84860fe0 T0)
    #0 0x774ad3b26fdb in __sanitizer::StackDepotBase<__sanitizer::StackDepotNode, 1, 20>::Put(__sanitizer::StackTrace, bool*) ../../../../src/libsanitizer/sanitizer_common/sanitizer_stackdepotbase.h:115
    #1 0x774ad3a42126 in __asan::Allocator::Allocate(unsigned long, unsigned long, __sanitizer::BufferedStackTrace*, __asan::AllocType, bool) ../../../../src/libsanitizer/asan/asan_allocator.cpp:609
    #2 0x774ad3a3ee30 in __asan::asan_memalign(unsigned long, unsigned long, __sanitizer::BufferedStackTrace*, __asan::AllocType) ../../../../src/libsanitizer/asan/asan_allocator.cpp:1059
    #3 0x774ad3afe501 in operator new(unsigned long) ../../../../src/libsanitizer/asan/asan_new_delete.cpp:95
    #4 0x5662dcb664d9 in Lexer::Lexer(XRef*, Stream*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Lexer.cc:53
    #5 0x5662dcbd6baf in XRef::fetch(int, int, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/XRef.cc:809
    #6 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106
    #7 0x5662dca96b8a in Dict::lookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Dict.cc:76
    #8 0x5662dcb70fc7 in Object::dictLookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.h:253
    #9 0x5662dcb79d46 in Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:156
    #10 0x5662dcb79692 in Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:94
    #11 0x5662dcbd6e45 in XRef::fetch(int, int, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/XRef.cc:823
    #12 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106
    #13 0x5662dca96b8a in Dict::lookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Dict.cc:76
    #14 0x5662dcb70fc7 in Object::dictLookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.h:253
    #15 0x5662dcb79d46 in Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:156
    #16 0x5662dcb79692 in Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:94
    #17 0x5662dcbd6e45 in XRef::fetch(int, int, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/XRef.cc:823
    #18 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106
    #19 0x5662dca96b8a in Dict::lookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Dict.cc:76
    #20 0x5662dcb70fc7 in Object::dictLookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.h:253
    #21 0x5662dcb79d46 in Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:156
    #22 0x5662dcb79692 in Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:94
    #23 0x5662dcbd6e45 in XRef::fetch(int, int, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/XRef.cc:823
    #24 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106
    #25 0x5662dca96b8a in Dict::lookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Dict.cc:76
    #26 0x5662dcb70fc7 in Object::dictLookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.h:253
    #27 0x5662dcb79d46 in Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:156
    #28 0x5662dcb79692 in Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:94
    #29 0x5662dcbd6e45 in XRef::fetch(int, int, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/XRef.cc:823
    #30 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106
    #31 0x5662dca96b8a in Dict::lookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Dict.cc:76
    #32 0x5662dcb70fc7 in Object::dictLookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.h:253
    #33 0x5662dcb79d46 in Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:156
    #34 0x5662dcb79692 in Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:94
    #35 0x5662dcbd6e45 in XRef::fetch(int, int, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/XRef.cc:823
    #36 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106
    #37 0x5662dca96b8a in Dict::lookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Dict.cc:76
    #38 0x5662dcb70fc7 in Object::dictLookup(char*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.h:253
    #39 0x5662dcb79d46 in Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:156
    #40 0x5662dcb79692 in Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Parser.cc:94
    ...
    #246 0x5662dcb6f1b3 in Object::fetch(XRef*, Object*) /home/cks/fuzzing/fuzzing_xpdf/xpdf-3.02/xpdf/Object.cc:106

SUMMARY: AddressSanitizer: stack-overflow ../../../../src/libsanitizer/sanitizer_common/sanitizer_stackdepotbase.h:115 in __sanitizer::StackDepotBase<__sanitizer::StackDepotNode, 1, 20>::Put(__sanitizer::StackTrace, bool*)
==460331==ABORTING
```

스택 트레이스를 보면 ASAN에 레드존에 닿아서 스택오버플로우가 발생했다고 나오는 것을 알 수 있다. (프레임 번호가 높을수록 먼저 실행된 Caller입니다.)

#### 무한 재귀가 반복되는 구간

스택 트레이스에서 **프레임 #5부터 #40까지**를 살펴보면, 정확히 6개의 함수가 한 세트가 되어 계속해서 꼬리를 물고 호출하고 있습니다. 재귀 사이클은 다음과 같이 구성되어 있습니다:

1. `XRef::fetch` (프레임 #5, #11, #17, #23, #29, #35...)
2. `Object::fetch` (프레임 #6, #12, #18, #24, #30, #36...)
3. `Dict::lookup` (프레임 #7, #13, #19, #25, #31, #37...)
4. `Object::dictLookup` (프레임 #8, #14, #20, #26, #32, #38...)
5. `Parser::makeStream` (프레임 #9, #15, #21, #27, #33, #39...)
6. `Parser::getObj` (프레임 #10, #16, #22, #28, #34, #40...)
7. **그리고 다시 1번 `XRef::fetch`로 돌아감**

`Parser::getObj`->`XRef::fetch`순으로 무한반복하는 것을 알 수 있습니다.

```shell
pwndbg --args ./install_asan_zero/bin/pdftotext ./out/default/crashes/id:000000,sig:11,src:000001,time:235035,execs:137919,op:havoc,rep:8
```
`pwndbg`을 통해 분석을 시작해본다. (다른 툴을 사용해도 된다.)

```cpp
// Object.cc:106
Object *Object::fetch(XRef *xref, Object *obj) {
  return (type == objRef && xref) ?
         xref->fetch(ref.num, ref.gen, obj) : copy(obj);
}
```
`Object.cc` 코드 일부를 보면 reference(xref)라는 단어가 있는 것을 알 수 있다. 스택오버플로우가 발생한 이유가 무한 자기 참조 때문에 난게 아닐까라고 의심할 수 있다. 그리고 gemini한테  해당 함수(`XRef` \: :`fetch(int num, int gen, Object *obj)`)에 대해 물어보니까. PDF의 객체 번호와 생성 번호를 받아서 객체를 가져오는 함수라고 했습니다.

```
pwndbg> b XRef::fetch
```
bp를 건 후 continue를 반복하다보면 num이 7을 반복합니다.

```
pwndbg> print num
$24 = 7
pwndbg> print gen
$25 = 0
```

텍스트 에디터로 크래시 파일을 연 후 `7 0 obj`를 검색해보면 아래 같은 부분이 나옵니다.

```
>>
/Contents 7 0 R
>>
endobj
7 0 obj
<</Length 7 0 R/Filter /FlateDecode>>
stream
```

발견하신 PDF 내용을 설명해보면 다음과 같습니다.
- `7 0 obj`: "7번 객체다."
- `<< /Length 7 0 R /Filter /FlateDecode >>`: "압축된 데이터(스트림)이 들어 있으므로 이 데이터의 길이(`/Length`)를 알고싶다면 7번 객체를 참조(`7 0 R`)해라!"



### 1. 스트림 생성 (`Parser`\:\:`getObj`)

`Parser::getObj(Object*, unsigned char*, CryptAlgorithm, int, int, int)`
`Parser.cc:94`
```cpp
if ((str = makeStream(obj, fileKey, encAlgorithm, keyLength,
          objNum, objGen)))
```
파서가 7번 객체를 읽다가 `<</Length ...>>` 뒤에 스트림 데이터가 있는 것을 발견합니다. 스트림 객체를 메모리에 생성하기 위해 `makeStream` 함수를 호출합니다.

#### 2. 스트림의 길이 확인 (`Parser`:\:`makeStream`)

`Parser::makeStream(Object*, unsigned char*, CryptAlgorithm, int, int, int)`
`Parser.cc:156`
```cpp
dict->dictLookup("Length", &obj);
```
스트림을 파싱하려면 이 데이터가 도대체 몇 바이트인지 알아야 합니다. 파서는 객체의 딕셔너리(`dict`)에서 `"Length"`라는 키(key)를 찾아 그 값(value)을 `obj` 변수에 담아오라고 지시합니다.

#### 3. 딕셔너리에서 값을 찾아라 (`Object`\:\:`dictLookup` -> `Dict`\:\:`lookup`)

`Object.h:253`
`Object::dictLookup(char*, Object*)`
`Dict.cc:76`
`Dict::lookup(char*, Object*)`
```cpp
// Object.h:253
inline Object *Object::dictLookup(char *key, Object *obj)
{ return dict->lookup(key, obj); }

// Dict.cc:76
return (e = find(key)) ? e->val.fetch(xref, obj) : obj->initNull();
```
딕셔너리에서 `"Length"` 키를 찾았습니다. 그런데 그 값(`e->val`)이 단순한 숫자(예: 1024)가 아니라 `7 0 R` 이라는 Reference 형태로 들어있습니다. 파서는 이 참조된 값을 실제 데이터로 변환해서 가져오기 위해 `fetch` 함수를 호출합니다

#### 4. 참조된 객체가 맞다면 XRef->fetch 호출 (`Object`:\:`fetch`)

`Object.cc:106`
`Object::fetch(XRef*, Object*)`
```cpp
Object *Object::fetch(XRef *xref, Object *obj) {
  return (type == objRef && xref) ?
         xref->fetch(ref.num, ref.gen, obj) : copy(obj);
}
```
여기서 치명적인 분기가 발생합니다. 현재 `Length`의 값인 `7 0 R`은 타입이 `objRef`(객체 참조)입니다. 따라서 `type == objRef` 조건이 참(True)이 됩니다. 자신이 직접 값을 줄 수 없으니, 상호 참조 테이블을 관리하는 `XRef` 객체에게 7번 객체(`ref.num`)를 넘깁니다.

#### 5. 새로운 파서를 열어 객체를 fetch (`XRef`:\:`fetch`)
`XRef.cc:809`
`XRef::fetch(int num, int gen, Object *obj)`
```cpp
parser = new Parser(this,
         new Lexer(this,
     str->makeSubStream(start + e->offset, gFalse, 0, &obj1)),
         gTrue);
```
`XRef`는 7번 객체를 파일에서 읽어오기 위해 **새로운 `Parser` 객체를 동적 할당(`new Parser`)하여 생성**합니다. 

#### 6단계: 무한 반복

새로 생성된 파서는 7번 객체를 처음부터 다시 읽기 시작합니다. "스트림이 있네. 길이를 알아야겠다." -> `"Length"를 찾아라` -> `어? 7 0 R 이네?` -> `XRef야 7번 객체 좀 가져와 줘` -> 또 다른 새로운 Parser 생성

결국 길이를 알아내려던 최초의 목적은 달성하지 못한 채, `Parser 생성 -> Length 확인 -> 참조 확인 -> Parser 생성` 이라는 굴레에 빠져 파서 객체만 무한정 생성하다가 스택 메모리가 초과되어 프로그램이 죽게 됩니다.


## 패치코드
---
### 1. `Object.h` 및 `Object.cc` 수정

객체를 가져올 때 재귀 깊이를 추적할 수 있도록 `recursion` 파라미터를 추가합니다.
(수정을 할려면 recursionDepth가 1씩 증가하며 함수 체인에 recursionDepth가 계속 전달되도록 수정해야 된다.)


#### **수정 전 (`Object.cc:106`, `Object.h:114`)**

```cpp
Object *Object::fetch(XRef *xref, Object *obj) {
  return (type == objRef && xref) ?
         xref->fetch(ref.num, ref.gen, obj) : copy(obj);
}
```

#### **수정 후 (`Object.cc:106`, `Object.h:114`)**

```cpp
// Object.h 의 선언부도 Object *fetch(XRef *xref, Object *obj, int recursionDepth = 0); 로 변경해야 합니다.

Object *Object::fetch(XRef *xref, Object *obj, int recursionDepth) {
  return (type == objRef && xref) ?
         // XRef::fetch를 호출할 때 현재 재귀 깊이를 그대로 전달합니다.
         xref->fetch(ref.num, ref.gen, obj, recursion) : copy(obj);
}
```

### 2. `XRef.h` 및 `XRef.cc` 수정 (핵심 방어 로직 추가)
---
실제로 무한 루프가 발생하는 `XRef::fetch`에 도달했을 때, 카운트를 검사하고 한계치를 넘으면 방어하는 로직을 추가합니다.



#### **수정 전 (`XRef.cc:809`, `XRef.h:75`)**

```cpp
Object *XRef::fetch(int num, int gen, Object *obj) {
  // ... 생략 ...
  parser = new Parser(this,
           new Lexer(this,
           str->makeSubStream(start + e->offset, gFalse, 0, &obj1)),
           gTrue);
  // ... 생략 ...
}
```
#### **수정 후 (`XRef.cc:809`, `XRef.h:75`)**

```cpp
#define MAX_RECURSION_DEPTH 100 // maximum recursion depth for fetching

Object *XRef::fetch(int num, int gen, Object *obj) {
	if (recursion > MAX_RECURSION_DEPTH) { return obj->initNull(); } // / 추가 로직 : 에러를 출력하거나 안전하게 Null 객체를 반환하여 루프를 끊어냅니다.

  // ... 생략 ...
  parser = new Parser(this,
           new Lexer(this,
           str->makeSubStream(start + e->offset, gFalse, 0, &obj1)),
           gTrue);
  // ... 생략 ...
}
```


## Conclusion
---
1. Fuzzing하기 전에 입력으로 사용하는 파일(pdftotext의 경우`pdf`)의 구조를 알아라.
2. 소스코드 주변을 간단하게 분석해라.
3. 소스 코드 하나하나의 동작을 전부 이해하기보다는 `Data Flow`와 이 함수가 도대체 무엇을 처리하고 어떤 기능을 수행하는지에 집중하여 핵심 변수를 모니터링해보자.


## Reference
---
https://github.com/antonio-morales/Fuzzing101/tree/main/Exercise%201
