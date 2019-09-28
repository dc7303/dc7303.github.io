---
layout: post
title: "[Python]메모리 관리"
subtitle: Reference Counts? Python GC?
date: 2019-08-06 10:00:00
background: '/img/posts/06.jpg'
category: python
tag: python, gc, ceference counting
comments: true
---
### 들어가기 앞서
인스타그램이 GC를 사용하지 않은 뒤 성능이 10%정도 향상 됐다는 글을 봤습니다. GC가 어떻게 동작하는지 궁굼하기도 했고 평소 파이썬으로 프로그래밍 하는 걸 좋아하기 때문에(필자는 자바 개발자..) 깊게 이해해보고 싶어 정리하게 됐습니다.

### 파이썬은 메모리 관리를 어떻게 하는가?
파이썬은 C 또는 C++과 같이 프로그래머가 직접 메모리를 관리하지 않고 레퍼런스 카운트(Reference Counts)와 가비지 콜렉션(Automatic Garbage Collection)에 의해 관리됩니다.

### 레퍼런스 카운트(Reference Counts)
파이썬은 내부적으로 malloc()와 free()를 많이 사용하기 때문에 메모리 누수의 위험이 있습니다. 이런 이슈가 있기 때문에 파이썬은 메모리를 관리하기 위한 전략으로 레퍼런스 카운트를 사용합니다.

레퍼런스 카운트 전략이란 파이썬의 모든 객체에 카운트를 포함하고, 이 카운트는 객체가 참조될 때 증가하고, 참조가 삭제될 때 감소시키는 방식으로 작동됩니다. 이때 카운터가 0이 되면 메모리가 할당이 삭제됩니다.

아래에 예제를 통해 살펴보겠습니다. sys 라이브러리의 getrefcount()를 통해 파라미터로 전달된 객체의 레퍼런스 카운트 확인할 수 있습니다.

```python
import sys

class RefExam():
  def __init__(self):
    print('create object')

a = RefExam()
print(f'count {sys.getrefcount(a)}')
b = a
print(f'count {sys.getrefcount(a)}')
c = a
print(f'count {sys.getrefcount(a)}')
c = 0
print(f'count {sys.getrefcount(a)}')
b = 0
print(f'count {sys.getrefcount(a)}')

"""
OUT PUT:
count 2
count 3
count 4
count 3
count 2
"""
```

위 코드를 보면 생성 직후 레퍼런스 카운트 2가 출력되는 걸 확인할 수 있습니다. 여기서 2가 출력되는 이유는 getrefcount()의 파라미터값으로 임시 참조되기 때문에 예상과 다르게 1이 아닌 2가 출력됩니다.([refcount](https://docs.python.org/3/library/sys.html#sys.getrefcount))

첫 번째 출력 이후 b, c에 각각 참조될 때 마다 1씩 증가하는 걸 확인할 수 있습니다. 그리고 b, c에 0이 할당될 때 1씩 감소하는 것을 확인할 수 있습니다.

이런 동작을 파이썬에서는 Py_INCREF와 Py_DECREF을 통해 구현되어 있습니다.[*(Python Doc)*](https://docs.python.org/ko/3.6/c-api/refcounting.html#c.Py_XINCREF) 

아래 코드는 Py_INCREF와 Py_DECREF을 통해 파이썬 내부에서 어떤 동작을 하는지 직관적으로 알 수 있습니다.

코드는 [cpython/object.h](https://github.com/python/cpython/blob/master/Include/object.h)에서 참고하였습니다.

```cpp
/* 파이썬의 객체 형태 */
typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;               /* 레퍼런스 카운트 */
    struct _typeobject *ob_type;
} PyObject;



/* ob_refcnt를 증가시킵니다. */
static inline void _Py_INCREF(PyObject *op)
{
    _Py_INC_REFTOTAL;
    op->ob_refcnt++;
}

/* ob_refcnt 0일때 _Py Dealloc(op)을 사용하여 메모리 할당을 제거합니다. */
static inline void _Py_DECREF(const char *filename, int lineno,
                              PyObject *op)
{
    (void)filename; /* may be unused, shut up -Wunused-parameter */
    (void)lineno; /* may be unused, shut up -Wunused-parameter */
    _Py_DEC_REFTOTAL;
    if (--op->ob_refcnt != 0) {
#ifdef Py_REF_DEBUG
        if (op->ob_refcnt < 0) {
            _Py_NegativeRefcount(filename, lineno, op);
        }
#endif
    }
    else {
        _Py_Dealloc(op);
    }
}
```

코드를 보면 [PyObject](https://github.com/python/cpython/blob/master/Include/object.h#L104)(파이썬객체)에서 ob_refcnt(레퍼런스 카운트) 프로퍼티를 가지고 있습니다. 그리고 [_Py_INCREF](https://github.com/python/cpython/blob/master/Include/object.h#L456)에서 증가시키고, [_Py_DECREF](https://github.com/python/cpython/blob/master/Include/object.h#L464)에서 감소시킨 후 레퍼런스 카운트가 0일 때 메모리를 할당을 삭제하는 _Py_Dealloc(op)를 실행하는 걸 확인할 수 있습니다. 

이렇게 파이썬은 기본적으로 레퍼런스 카운트를 통해 메모리를 관리합니다. 레퍼런스 카운트는 메모리 관리에 매우 효율적으로 동작하지만, 레퍼런스 카운트만으로 메모리를 관리했을 때 약점이 있습니다.

### 순환 참조
순환 참조란 간단하게 컨테이너 객체가 자기 자신을 참조하는 것을 말합니다. 자기 자신이 참조될 때 프로그래머는 할당된 객체를 추적하기 어려워지고, 이때 메모리 누수가 발생할 수 있습니다. 아래 예제 코드를 보겠습니다.


```python
class RefExam():
  def __init__(self):
    print('create object')
  def __del__(self):
    print(f'destroy {id(self)}')


a = RefExam()
a = 0
print('end .....')

"""
OUT PUT:
create object
destroy 3112733520336
end .....
"""
```

\__del__()은 메모리 할당이 삭제되는 시점에 실행되는 메소드입니다. 위 코드와 같이 a 변수에 0을 재할당할 때 \__del__()이 실행되고 마무리하는 것을 확인할 수 있습니다. 하지만 여기서 순환 참조가 될 때는 아래 코드 예제처럼 출력됩니다.

```python
# me 프로퍼티에 자기 자신을 할당합니다.
class RefExam():
  def __init__(self):
    print('create object')
    self.me = self
  def __del__(self):
    print(f'destroy {id(self)}')

a = RefExam()
a = 0
print('end .....')

"""
OUT PUT:
create object
end .....
destroy 2110595412432
"""
```

위 코드는 자기 자신을 할당하기 전과 다르게 'end .....'를 출력하고 \__del__()이 실행되는 걸 확인할 수 있습니다.

예제에서 볼 수 있듯이 a 변수에 새로운 값을 할당해도 a.me 프로퍼티에 자기 자신을 참조하고 있어 레퍼런스 카운트가 남아있기 때문에 이런 현상이 발생합니다.

이렇게 되면 레퍼런스 카운트가 0에 도달할 수 없고 할당된 메모리를 삭제할 수 없어 메모리 누수가 발생합니다.

파이썬은 이 문제를 가비지 콜렉션으로 해결합니다. 
















### 가비지 콜렉션(Automatic Garbage Collection)
설명에 들어가기에 앞서 알아두셔야 하는 것은 레퍼런스 카운트도 가비지 콜렉션이(GC)라고 부릅니다. 이를 구분하기 위해서 순환 참조 이슈를 해결하기 위해 구현한 가비지 콜렉션을 'Automatic garbage collection'이라고 부릅니다. ([Python Doc 1.10.Reference Counts](https://docs.python.org/ko/3/extending/extending.html#reference-counts))

파이썬에서는 [Cyclic Garbage Collection](https://docs.python.org/3/c-api/gcsupport.html)을 지원합니다. 이는 순환 참조 이슈를 해결하기 위해 존재하며, 참조 주기를 감지하여 메모리 누수를 예방합니다.

### Generational Hypothesis
가비지 콜렉션은 Generational Hypothesis라는 가설을 기반으로 작동합니다. 이 가설은 '대부분의 객체는 생성되고 오래 살아남지 못하고 곧바로 버려지는 것'과 '젊은 객체가 오래된 객체를 참조하는 상황은 드물다'는 2가지 가설입니다.

이 가설을 기반으로 메모리에 존재하는 객체를 오래된 객체(old)와 젊은 객체(young)로 나눌 수 있는데, 대부분의 객체는 생성되고 곧바로 버려지기 때문에 젊은 객체에 비교적 더 많이 존재한다고 볼 수 있습니다.

즉, Generational Hypothesis를 기반으로 작동한다는 것은 젊은 객체에 대부분의 객체가 존재하니, 가비지 컬렉터가 작동 빈도수를 높여 젊은 객체 위주로 관리해주는 것입니다.

### 세대관리
파이썬은 객체 관리를 위한 영역을 3가지로 나뉩니다. 이 영역을 세대(generation)라고 합니다. 파이썬에서 세대를 초기화할 때 아래의 [_PyGC_Initialize](https://github.com/python/cpython/blob/bf8162c8c45338470bbe487c8769bba20bde66c2/Modules/gcmodule.c#L129) 메소드를 호출하는데 3세대를 초기화하는 걸 확인할 수 있습니다.


```cpp
#define NUM_GENERATIONS 3                 /* 3세대로 관리 */

// ...

#define GEN_HEAD(state, n) (&(state)->generations[n].head)

// ...

void
_PyGC_Initialize(struct _gc_runtime_state *state)
{
    state->enabled = 1; /* automatic collection enabled? */

  #define _GEN_HEAD(n) GEN_HEAD(state, n)
    struct gc_generation generations[NUM_GENERATIONS] = {
        /* PyGC_Head,                                           threshold,    count */
        \{\{(uintptr_t)_GEN_HEAD(0), (uintptr_t)_GEN_HEAD(0)\},   700,        0\},      /** 0세대 초기화 */
        \{\{(uintptr_t)_GEN_HEAD(1), (uintptr_t)_GEN_HEAD(1)\},   10,         0\},      /** 1세대 초기화 */
        \{\{(uintptr_t)_GEN_HEAD(2), (uintptr_t)_GEN_HEAD(2)\},   10,         0\},      /** 2세대 초기화 */
    };
    for (int i = 0; i < NUM_GENERATIONS; i++) {
        state->generations[i] = generations[i];
    };
    
  // ...
}
```

코드를 초기화할 때 임계 값(threshold)을 각 700, 10, 10으로 초기화하고 카운트(count)를 0, 0, 0으로 초기화합니다.

파이썬 런타임 환경에서 설정된 임계 값과 현재 카운트를 확인을 gc.get_threshold()와 gc.get_count()로 확인할 수 있습니다.


```python
import gc
print(gc.get_threshold())
print(gc.get_count())
"""
OUTPUT:
(700, 10, 10)
(18, 7, 8)              // 현재 count상태를 확인하는 것이기 때문에 출력값이 다를 수 있다.
```

파이썬은 이렇게 셋팅한 세대별 임계값과 할당된 카운트를 비교하여 콜렉션을 결정합니다.

임계값을 활용하는 방법은 객체가 생성될 때 0세대의 카운트 값이 증가합니다. 증가될 때 0세대의 카운트와 임계값을 비교하여 만약 카운트가 임계 값보다 클 때 쓰레기 수집을 실행하고 0세대는 초기화됩니다.

0세대의 살아남은 객체는 다음 1세대로 옮겨지고 1세대의 카운트(count)는 1 증가합니다. 이런 방식으로 젊은 세대(young)에서 임계 값이 초과하면 오래된 세대(old)로 위임하는 방식으로 3세대 영역으로 관리됩니다.

### 좀 더 자세히
파이썬은 객체가 생성될 때 [_PyObject_GC_Alloc](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L1934) 메소드를 호출합니다.

```cpp

static PyObject *
_PyObject_GC_Alloc(int use_calloc, size_t basicsize)
{
    struct _gc_runtime_state *state = &_PyRuntime.gc;
    PyObject *op;
    PyGC_Head *g;
    size_t size;
    if (basicsize > PY_SSIZE_T_MAX - sizeof(PyGC_Head))                         /* 메모리 할당 */
        return PyErr_NoMemory();
    size = sizeof(PyGC_Head) + basicsize;
    if (use_calloc)
        g = (PyGC_Head *)PyObject_Calloc(1, size);
    else
        g = (PyGC_Head *)PyObject_Malloc(size);
    if (g == NULL)
        return PyErr_NoMemory();
    assert(((uintptr_t)g & 3) == 0);  // g must be aligned 4bytes boundary
    g->_gc_next = 0;
    g->_gc_prev = 0;
    state->generations[0].count++; /* number of allocated GC objects */         /* 0세대 증가 */
    if (state->generations[0].count > state->generations[0].threshold &&        /* 임계값 비교 */
        state->enabled &&                                                       /* 사용여부 */
        state->generations[0].threshold &&                                      /* 임계값 설정 여부 */
        !state->collecting &&                                                   /* 수집중 여부 */
        !PyErr_Occurred()) {

        state->collecting = 1;                                                  /* 수집 상태 활성화 */
        collect_generations(state);                                             /* 모든 세대 검사 메소드 */
        state->collecting = 0;
    }
    op = FROM_GC(g);
    return op;
}

```

[_PyObject_GC_Alloc](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L1934) 코드를 보면 generations[0].count (0세대)를 증가시킵니다. 이후 현재 상태를 확인(우측 주석 확인)하여 조건에 충족하지 않을 때 [collect_generations](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L1229)을 호출합니다.

```cpp
static Py_ssize_t
collect_generations(struct _gc_runtime_state *state)
{
    Py_ssize_t n = 0;
    for (int i = NUM_GENERATIONS-1; i >= 0; i--) {                            /** 마지막 세대부터 확인 */
        if (state->generations[i].count > state->generations[i].threshold) {
            if (i == NUM_GENERATIONS - 1
                && state->long_lived_pending < state->long_lived_total / 4)   /** 새 객체 수가 기존 객체 수의 25%를 초과하면 전체 콜렉션 실행 */
                continue;
            n = collect_with_callback(state, i);
            break;
        }
    }
    return n;
}
```

코드에서 보는 것처럼 오래된 세대부터 젊은 세대로 내려오면서 임계 값을 검사합니다.

여기서 마지막 세대 INDEX를 가질 때 **state->long_lived_pending**과 **state->long_lived_total / 4**를 비교하는 걸 볼 수 있습니다.

이는 가비지 컬렉션 성능 향상을 위한 전략으로 새로 생성된 객체(long_lived_pending)의 수가 기존의 살아남았던 객체(long_lived_total)의 25%를 기준으로 기준치를 초과했을 때 전체 콜렉션이 실행됩니다. 자세한 내용은 [pycore_pymem.h](https://github.com/python/cpython/blob/master/Include/internal/pycore_pymem.h#L22)문서의 NOTE 주석을 통해 확인할 수 있습니다.

조건문 조건에 만족하는 세대를 [collect_with_callback](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L1217) 호출에 파라미터값으로 전달합니다.

```cpp
static Py_ssize_t
collect_with_callback(struct _gc_runtime_state *state, int generation)
{
    assert(!PyErr_Occurred());
    Py_ssize_t result, collected, uncollectable;
    invoke_gc_callback(state, "start", generation, 0, 0);
    result = collect(state, generation, &collected, &uncollectable, 0);
    invoke_gc_callback(state, "stop", generation, collected, uncollectable);
    assert(!PyErr_Occurred());
    return result;
}
```

[collect_with_callback](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L1217) 함수에서 GC의 핵심인 [collect](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L987)를 호출합니다. [collect](https://github.com/python/cpython/blob/master/Modules/gcmodule.c#L987)는 내부에서 콜렉션 대상 이하의 세대 카운트를 초기화하고, 도달 가능(reachable)한 객체와 도달할 수 없는(unreachable) 객체를 분류합니다. 그리고 분류된 도달할 수 없는 객체들을 메모리에서 삭제합니다.


이 과정은 먼저 레퍼런스 카운트(RC)를 gc_refs에 복사합니다. 그리고 객체에서 참조하고 있는 다른 컨테이너 객체를 찾아 참조되고 있는 컨테이너 객체의 gc_refs를 감소시킵니다. (*TIP. 순환 참조는 컨테이너 객체에서 발생할 수 있는 이슈입니다*)


즉, 다른 컨테이너 객체에 참조되고 있는 수 A와 현재 레퍼런스 카운트 B를 빼서 B - A > 0 일 경우 도달 가능한 객체(reachable)가 되고, 0 일 때 도달할 수 없는 객체(unreachable)로 분류합니다.


이후 도달 가능한 객체들은 다음 세대 리스트와 병합되고, 도달할 수 없는 객체들은 메모리에서 제거됩니다. 이런 메커니즘을 **순환 참조 알고리즘**이라고 합니다.

### 컨테이너 순회는 어떻게 할까?

가비지 컬렉션이 발생할때 컨테이너들을 순회할 수 있는 이유는 파이썬에서 관리되는 각 [gc_generation](https://github.com/python/cpython/blob/master/Include/internal/pycore_pymem.h#L97)[N]은 [PyGC_Head]((https://github.com/python/cpython/blob/master/Include/cpython/objimpl.h#L46)) 타입의 head 프로퍼티를 포함하고 있습니다. head는 아래 코드와 같이 구현되어 있습니다.


```cpp
typedef struct {
    // Pointer to next object in the list.
    // 0 means the object is not tracked
    uintptr_t _gc_next;

    // Pointer to previous object in the list.
    // Lowest two bits are used for flags documented later.
    uintptr_t _gc_prev;
} PyGC_Head;
```

코드를 보면 [PyGC_Head]((https://github.com/python/cpython/blob/master/Include/cpython/objimpl.h#L46))는 더블 링크드 리스트를 구현하고 있는 구조체입니다. 이를 통해 세대에 포함된 컨테이너 객체들을 순회할 수 있게 됩니다.

### 마무리하며
파이썬의 메모리 관리를 정리하는데 큰 재미를 느꼈습니다. 메모리를 관리해주는 언어들을 여럿 사용할때 이론적인 부분만 이해하고 있었는데, 이렇게 실제 코드를 따라가며 분석해보니 추상적인 이해에서 더 구체적으로 이해할 수 있어서 좋았습니다.

그리고 오픈 소스를 분석한다는 의미에서 파이썬 개발팀이 어떤 코드를 지향하는지 조금이나마 들여다 볼 수 있어 종핬습니다. 종종 파이썬을 사용하면서 궁굼한 부분들을 직접 들여다 보며 분석하는 시간을 가져야겠습니다.

아쉬운 점은 collect가 호출되는 부분을 이 글에 자세히 설명하기에는 내용이 너무 길어질 것 같아 메커니즘 설명으로만 정리하였습니다. 추후 시간을 내서 관련 글을 정리할 생각입니다.

### Reference
 - [Java Garbage Collection](https://d2.naver.com/helloworld/1329)
 - [Instagram이 Python garbage collection 없앤 이유](https://b.luavis.kr/python/dismissing-python-garbage-collection-at-instagram)
 - [cpython](https://github.com/python/cpython)
 - [Garbage Collector interface](https://docs.python.org/3/library/gc.html)
 - [Extending Python with C or C++](https://docs.python.org/ko/3/extending/extending.html)
 - [Python GC가 작동하는 원리](https://winterj.me/python-gc/)