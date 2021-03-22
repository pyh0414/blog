---
title: '[Git] merge와 rebase'
date: 2021-03-16 19:03:96
category: git
thumbnail: { thumbnailSrc }
draft: false
---

입사하고 git의 부족함을 느껴 몇일동안 퇴근하고 git 공부를 많이 했는데, 그 중에 가장 많이 공부했던 부분이 merge와 rebase 입니다. 회사에서 rebase와 merge를 같이 사용하는데, 기능 하나하나 정확하게 알겠다 보다는 각각의 상황에 맞게 rebase와 merge를 사용하는 방법과 기본적인 개념에 대해 공부했습니다.

# merge와 rebase 대충 살펴보기

**merge와 rebase는 한 브랜치에서 다른 브랜치로 합치는 방법입니다.**  
어쨋든 두 방법의 최종적인 목적은 코드를 합치는 것입니다. 다른점은 어떻게 합치는지만 다를 뿐입니다. 본격적으로 알아보기 전에 각 단어의 의미를 톺아봅시다.

### 1. merge

합병하다, 합치다, (서로 구분이 안 되게)어우러지다.

### 2. rebase

새로운 평가(산정) 기준을 설정하다  
base(맨 아래 부분, 기초, 토대)를 re(다시 만든다, 설정한다)한다.

🤔 merge의 경우 두개의 브랜치를 합치는 것 같습니다. 반면에 rebase는 의미가 좀 애매한데 기초를 다시 설정한다는 의미 같습니다. 일단은 이정도로만 알고 가겠습니다.

# merge

각 브랜치의 커밋들을 `시간순`으로 합치고 merge commit을 하나 생성합니다.

branch2위치에서 git merge branch1을 하면 branch1의 내용을 branch2에게 merge한다, 붙인다, 합친다 라고 생각할 수 있습니다. merge된 후 merge했음을 기록하는 commit이 branch2에 생깁니다.

branch1에서의 commit log는 이전과 달라진게 없지만 branch2에서의 commit log는 branch1의 내용과 branch2의 내용이 시간순서로 합쳐지고 마지막에 merge commit이 생성됩니다.

```javascript
  branch1 ─┬──────────────────────────────────────────────── ...
           │              ↑                  ↑
           │            s71i🔴             z72u🔴
           │            21:32              21:48
           │
           │
  branch2  └─────────────────────────────────────────────── ...
               ↑                   ↑
             a45g🟢              g80z🟢
             21:24               21:41


                🔀 branch2에서 git merge branch1

                               ⬇

  branch1 ─┬────────────────────────────────────────────┐
           │              ↑                 ↑           │
           │            s71i🔴             z72u🔴        │
           │            21:32              21:48        │
           │                                            │
           │                                            ↓
  branch2  └─────────────────────────────────────────────── ...
               ↑          ↑         ↑         ↑        ↑
              a45g🟢    s71i🔴    g80z🟢     z72u🔴    merge🟡
              21:24     21:32     21:41     21:48    21:57
```

&nbsp;

# rebase

rebase는 이해하기가 좀 난해합니다. 저역시 rebase의 동작을 이해하는데 시간이 좀 걸렸는데요, 결국에는 rebase도 merge처럼 두 브랜치를 합치는(merge하는) 방법중 하나입니다

rebase는 base를 re한다, 즉 기초를 다시 설정합니다.

```javascript
  branch1 ─┬──────────────────────────────────────────────── ...
           │              ↑                  ↑
           │            s71i🔴             z72u🔴
           │            21:32              21:48
           │
           │
  branch2  └────────────────────────────────────────────────  ...
               ↑                   ↑
             a45g🟢              g80z🟢
             21:24               21:41

                🔀 branch2에서 git rebase branch1

                               ⬇

  branch1 ─┬──────────────────────────────────────────────── ...
           │              ↑                 ↑
           │            s71i🔴            z72u🔴
           │            21:32             21:48
           │
           │
  branch2  └──────────────────────────────────────────────── ...
               ↑          ↑         ↑         ↑
              s71i🔴    z72u🔴     k23y🟢    j28u🟢
              21:32     21:48     21:24     21:41
```

rebase의 경우 branch1의 commit내용이 branch2의 commit 발 아래로 들어왔습니다(저는 발 아래로 표현하는 것이 이해하는데, 어떤 방법으로든 이해하면 됩니다). 먼저 branch2의 commit을 살펴보면 `a45g🟢` `g80z🟢` 2개가 있습니다. 이 상황에서 branch2의 base를 branch1로 한다는 것은 branch1의 commit을 `a45g🟢` `g80z🟢` 보다 더 과거에 놔두겠다는 것을 의미합니다.

이렇게 되면 `s71i🔴` `z72u🔴` `k23y🟢` `j28u🟢` 순으로 branch2의 commit 다시 구성됩니다.

`레고쌓기`로 rebase를 생각해보면 각각의 브랜치는 2개의 레고가 있는데, 원래는 따로 쌓여 있다가 합치는 과정에서 branch1의 레고2개를 branch2 밑에 쌓는 것과 같다고 생각하시면 됩니다.

⚠️ 여기서 중요한 점은 rebase 이후 branch2의 기존 `commit의 hash값이 변경`된다는 것입니다. 이는 커밋 메세지가 같더라도 git은 아예 다른 commit으로 이해합니다.

### conflict로 rebase 동작 이해하기

branch2에서 git rebase branch1을 하면 branch1의 commit들이 base로 잡히고 그 위에 branch2의 commit들이 쌓이게 된다(여기서 `그 위에`라고 표현을 했는데, 이는 branch1의 commit들이 시간순으로 더 과거에 base로 잡히고 그 바로 다음 branch2의 commit들이 온다고 하여 `그 위에`라는 표현을 사용했습니다)

여기서 기존 branch2의 commit은 hash값이 변경되는데, 왜 변경되는지 rebase의 동작원리를 이해하면서 알아보겠습니다.

```javascript
  branch1 ─┬──────────────────────────────────────────────── ...
           │          ↑           ↑             ↑
           │        s71i🔴       z10i🔴        z72u🔴
           │
           │
           │
  branch2  └────────────────────────────────────────────────  ...
                      ↑
                    a45g🟢

```

branch1에서 git rebase branch2를 하면 branch1의 commit이 임시 공간에 저장되게 됩니다. 이후 임시 공간에서 commit을 하나씩 꺼내오면서 branch2의 commit과 합치고 새로운 commit 생성, 다시 임시공간에서 하나 꺼내와 branch2의 commit와 합치고 새로운 commit 생성.....의 과정을 거칩니다.

여기서 합치면서 commit이 새롭게 생성되기 때문에 `hash값이 변경`되는 것입니다.

⚠️ 단 여기서 a45g🟢은 branch1의 commit 내용과 merge되면서 단계가 진행되면서 내용이 수정됩니다.

&nbsp;

1. s71i🔴 + a45g🟢 = z12p🟡

```javascript
  branch1 ─┬────────────────────────────────────────────────── ...
           │    ↑       ↑         ↑      ↑
           │  s71i🔴   z10i🔴    z72u🔴  z12p🟡
           │
```

2. z10i🔴 + a45g🟢 = y90z🟡

```javascript
  branch1 ─┬────────────────────────────────────────────────── ...
           │    ↑       ↑         ↑      ↑        ↑
           │  s71i🔴   z10i🔴    z72u🔴  z12p🟡  y90z🟡
           │
```

3. z72u🔴 + a45g🟢 = g65h🟡

```javascript
  branch1 ─┬────────────────────────────────────────────────── ...
           │    ↑       ↑         ↑      ↑        ↑      ↑
           │  s71i🔴   z10i🔴    z72u🔴  z12p🟡  y90z🟡  g65h🟡
           │
```

4. 마지막

```javascript
  branch1 ─┬────────────────────────────────────────────────── ...
           │    ↑       ↑        ↑      ↑
           │  a45g🟢  z12p🟡  y90z🟡  g65h🟡
           │
```

여기서 중요한 점은 branch1의 commit내용을 branch2와 merge할 때마다 `새로운 commit이 생성`되고 branch1의 기존 commit은 유지되다가 `rebase작업이 끝날때 branch1의 기존 commit이 없어집니다.`

# 어떻게 다른가 ?

지금까지 merge와 rebase를 알아보았는데요(사실 rebase도 merge의 방법 중 하나 입니다), 이 둘의 차이점을 알아보면 아래와 같습니다.

1.`git log --decorate --all --graph --oneline`로 commit log를 보면 `merge는 두 commit 실선이 하나의 점에서` 만나지만 `rebase는 각기 다른 실선을 가집니다.`

2.merge는 두 실선이 하나의 점에서 만나기 때문에, merge의 흐름을 가시적으로 확인할 수 있습니다.

3.rebase는 branch를 따서 작업을 할 때 master에서 최신 pull 내용을 포함하면서 개발하고 싶을 때 사용할 수 있습니다. 또한 rebase가 한 줄로 표현되기 때문에 보다 깔끔합니다.

4.저희 팀에서는 develop, staging, master가 합쳐야 하는 경우 merge 사용합니다. 앞의 branch는 중요한 역할을 하기 때문에 merge를 하면 하나의 점선으로 모이기 때문에 가시적으로 확인할 수 있고, 대부분 version release를 담당하기 때문에 merge commit에 표시할 수 있습니다.

5.반면 branch에 작업한 내용은 develop에 합칠때는 rebase를 하는데, 이는 develop의 기능을 base로 하면서 branch의 작업내용을 그 위에 올리기 때문에 흐름상으로 적합합니다. 또한 불필요한 merge commit을 생성하지 않습니다.

## 📖 Reference

- <https://opentutorials.org/course/2708/15553>
