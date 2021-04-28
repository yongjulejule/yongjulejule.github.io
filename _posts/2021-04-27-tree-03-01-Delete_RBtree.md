---
layout: post
title: "Tree 03-2. Red-Black Tree (레드블랙 트리의 삭제)"
comments: true
excerpt_separator: "<!--more-->"
categories: 
- Algorithms
tags:
- Algorithms
- tree
- Red-Black Tree
order: 4
date: '2021-04-27 13:04:00 +0900'
last_modified_at: '2021-04-28 18:00:00 +0900'
---

> 이전 게시글에 이어서, Red-Black Tree의 삭제 부분을 알아보는 게시글 입니다. _Introduction to Algorithms_ 책을 참고하였습니다.
>
> 정리 하면서도 확실히 이해했다는 생각이 안들어 틀린 부분이 있을 수 있습니다 ㅠㅠ

<!--more-->

# Deletion of Red-Black Tree

레드블랙 트리의 삭제 역시 $O(lgn)$ 의 시간복잡도를 갖지만, 삽입보다 좀 더 복잡한 과정을 거쳐야 한다. 전체적인 과정은 BST의 삭제 과정과 비슷하지만, 레드블랙 트리의 성질에 맞춰주는 과정이 추가된다.

## RB-Transplant

```c
RB-Transplant(T, u, v) //u의 위치에 v를 넣어줌
if u.p == T.nil //u가 루트노드였으면, 루트를 바꿔준다.
  T.root = v;
else if u == u.p.left //왼쪽 or 오른쪽 자식 이었으면 그 자리에 새로운 v를 연결해준다. 
  u.p.left = v;
else u.p.right = v;
v.p = u.p
```

부모 / 자식 노드를 다시 맞춰주는 `Transplant` 과정은 BST와 거의 유사하며 이해하는데 무리가 없다.

## RB-Delete

BST에서 삭제 과정과 비슷하지만, `color` 를 맞춰주기 위한 과정이 많이 추가되었다. 

_(삭제하고싶은 노드 = `z`, 대체할 노드 = `y`, `y`의 자식 노드 = `x`)_

만약 삭제하고 싶은 노드 `z`의 자식 노드가 2개보다 적다면, `z`를 삭제하고 그 자식 노드`y`를 `z.parent`에 연결해주면 된다. 

`z`의 자식 노드가 2개라면, `z`의 `successor`(y)가 `z`의 위치에 오면 된다. 이때 원래 `y`의 자식 노드와 `y.color`가 레드블랙 properties를 위반할 수 있으므로 이를 생각해 두어야 하며, z를 삭제한 후 `RB-Delete-Fixup` 과정을 통해 레드블랙 트리의 형태로 만들어 준다.

### Pseudocode 

```c
RB-Delete(T,z)
y = z
y-original-color = y.color
if z.left == T.nil //z의 자식노드가 2개 미만일 경우
  x = z.right //5, 8, 13 라인에서 원래 y의 위치를 계속 x에 저장한다
  RB-Transplant(T,z,z.right)
else if z.right == T.nil
  x = z.left
  RB-Translpant(T,z,z.left)
else //z의 자식노드가 2개일 경우
  y = Tree-Minimum(z.right) //y는 z의 successor
  y-original-color = y.color
  x = y.right
  if y.p == z
    x.p = y
  else 
    RB-Transplant(T,y,y.right)
    y.right = z.right
    y.right.p = y
  RB-Transplant(T,z,y)
  y.left = z.left
  y.left.p = y
  y.color = z.color
if y-original-color == BLACK
  RB-Delete-Fixup(T,x)
```

- Line 9 까지는 `z`의 자식노드가 2개 미만이여서 바로 삭제하고 그 자식노드를 해당 위치로 옮겨주는 과정이며 원래 색깔을 `y-original-color`에 저장해 Line 24에서 레드블랙 트리의 성질을 위반했는지 확인한다.
- y의 원래 자식 노드를 기억해야 하므로 Line 5, 8, 13 에서 `x`에 저장해준다. 만약 `y`의 자식이 없다면, `x`는 `nil`이 될 것이다.
- `x`를 움직이면서, `x`의 부모 노드도 재설정 하는데, 이 과정을 Line 6, 9, 17 `RB-Transplant` 에서 수행해준다.
- Line 24에서 원래 `y`가 `BLACK` 이었으면 레드블랙트리의 성질을 위반하므로 `RB-Delete-Fixup`로 수정해준다. 만약 `y`가 `RED`라면 다음과 같은 이유로 수정하지 않아도 된다.
- cf) `z`를 삭제하고 들어간 `y` 자리는 색깔에 오류가 없음!! `y.color`에 `z.color`를 넣기 때문. 하지만 원래 `y`자리의 색깔을 고려하지 않고 `x`를 넣었기 때문에 이를 고려해줘야함!!! (Fixup에 `x`를 넣는 이유)

> 1. Black-heights가 변하지 않았다.
> 2. 인접한 노드에 `RED`노드가 만들어지지 않는다. -  왜냐하면 `y`는 `z`의 위치를 차지하는데, 이때 `z`의 색깔도 함께 가져오므로 이전에 트리가 레드블랙 properties를 만족하고 있었다면 연속된 `RED`가 나올 수 없다. 또한 만약 `y`가`z`의 `right`가 아니라면 원래 `y`의 `right` 인 `x`가 `y`의 자리에 온다. 만약 `y`가 `RED`라면 `x`는 `BLACK`이어야 하고 따라서 `y`를 `x`로 교체하는 것이 연속된 `RED`를 만들어 낼 수 없다.
> 3. `y`가 레드이기 때문에 `root`가 아니고, 따라서 `root`는 `BLACK`으로 잘 유지된다.

## RB-Delete-Fixup

> 참고) 레드블랙 트리의 Properties
>
> 1. 모든 노드는 `red` 이거나 `black` 이다.
> 2. `root` 노드는 `black` 이다.
> 3. 모든 `leaf(NIL)` 는 black이다.
> 4. 만약 한 노드가 `red` 이면 자식 노드(들)은 `black` 이다.
> 5. 각각 노드에서, 그 노드로부터 leaves 까지의 simple path들은 같은 수의 `black` 노드를 지난다.

만약 `y`가 `BLACK`이면 레드블랙트리의 성질을 위반하게 되고 이를 고쳐주기 위해 `RB-Delete-Fixup` 과정이 필요하다. 이때 다음과 같은 3가지의 문제가 발생한다.

1. 만약 `y`가 `root`였고, `y`의 `RED` child가 새로운 `root`가 된다면 Property 2를 위반하게 된다.
2. 만약 `x`와 `x.parent`가 `RED`이면 Property 4를 위반하게 된다.
3. `y`를 트리 내에서 이동하면서 이전에 `y`를 포함했던 Simple path는 `BLACK` 노드를 하나 덜 가지게 된다. 따라서 `y`의 ancestor 노드는 Property 5를 위반하게 된다. 

원래의 `y`자리를 차지하던 `x`가 "extra black"를 가진다고 생각해보자. 즉 만약 `x`를 포함하는 simple path에 하나의 `BLACK` 노드를 추가한다면 Property 5에 맞을 것이다. 결국 `BLACK` 노드 `y`를 제거하거나, 이동시킬때 `y`의 blackness를 `x`에 "Push" 하는 것이다. 

문제는 `x`는 `RED`도 `BLACK`도 아니게 되어 Property 1을 위반한다. 이때 `x`는 각각 "doubly black" 이나 "red-and-black" 으로 이들은 `x`를 포함하는 simple path의 `BLACK` 노드의 수에 +2나 +1을 해준다. `x`의 `color`은 여전히 `RED`(`x`가 red-and-black) 나 `BLACK`(`x`가 doubly black)이며 노드의 extra black 는 `x`의 컬러 특징보단 `x`가 가리키는 노드를 나타낸다. 

이해하기 상당히 난해한데, 쉽게 말하면 진짜 `BLACK`는 아닌데 `BLACK`의 성질을 띄는 extra black으로 `color`을 정하고 doubly-black라 부름. 이 doubly black는 black depth를 count할때 2로 카운트 하지만 레드블랙 트리에서 실제로 유효한 컬러가 아니기 때문에 fix를 해줘야함! ->왜 이런 짓을 하는가 싶지만, doubly black 노드를 추적하면 어디에 black를 저장할 자리가 있는지 알 수 있다는 이점이 있음!!!

### pseudo code

```c
RB-Delete-Fixup(T,x)
  while x != T.root and x.color == BLACK
    if x == x.p.left //w는 x의 형제노드
      w = x.p.right
      if w.color == RED //case 1
        w.color = BLACK
        w.p.color = RED
        leftRotate(T, x.p)
        w = x.p.right
      if w.left.color == BLACK and w.right.color == BLACK //case 2
        w.color = RED
        x = x.p
      else if w.right.color == BLACK
        w.left.color = BLACK //case 3
        w.color = RED
        rightRotate(T, w)
        w = w.p.right
      w.color = x.p.color //case 4
      x.p.color = BLACK
      w.right.color = BLACK
      leftRotate(T,x.p)
      x = T.root
    else (x가 오른쪽 자식일 경우. symmetric하게 바꿔주면 됨)
x.collor = BLACK
```

`RB-Delete-Fixup`는 Properties 1, 2, 4를 고쳐준다. 

Line 2부터 Line 23까지 `while`문의 목표는 다음과 같다. 

1. `x`가 red-and-black 노드를 가리켜서, Line 24에서 `BLACK`으로 만들어 줄 수 있거나
2. `x`가 `root`를 가리켜서, 단순히 extra black를 삭제시킬 수 있거나
3. 적절하게 회전과 recoloring을 해서 반복문을 빠져나간다.

이 반복문을 도는 동안 `x`는 항상 루트가 아닌 doubly-black 노드를 가리킨다. 이 `x`는 doubly black이기 때문에 `x`의 자매 노드인 `w`는 nil이 될 수 없다. 그렇지 않으면 `x.parent`(singly black)로부터 `leaf`인 `w`까지 simple path에서 `BLACK`의 수는 `x.parent` 부터 `x`까지 simple path의 `BLACK`의 수보다 작아지게 되기 때문이다.

![FIgure 0. whole image](https://github.com/chatsh1re/chatsh1re.github.io/blob/master/_posts/images/RB%20Tree/whole.png?raw=true)

여기서 4가지 case가 나오는데, 각 케이스들이 Property 5를 유지하는 방법을 살펴보고 어떻게 진행되는지 분석해보자. 

완전히 까맣게 색칠되어진 노드는 `BLACK`이고 진한 회색으로 칠해진 노드는 `RED`이다. 연한 회색으로 칠해진 노드는 `BLACK`이나 `RED` 둘 다 될 수 있는 노드이다. 각 $\alpha, \beta, ... ,\zeta$ 는 임의의 서브트리이다. x가 가리키는 노드는 extra black노드이며 doubly black나 red-and-black 이다. 

***Case 1: x의 자매노드 w가 RED***

![Figure 1. (a)](https://github.com/chatsh1re/chatsh1re.github.io/blob/master/_posts/images/RB%20Tree/case 1.png?raw=true)

*Case 1은 B와 D의 색깔을 서로 바꾸고 `leftRotate`를 하여 Case 2, 3, 4로 바꿔준다.*

> Property 5: 루트 노드로 부터 서브트리 $\alpha, \beta$  까지 `BLACK` 노드의 수는 3이고 (x는 doubly black!) $\gamma,\delta,\zeta$ 까지 `BLACK` 노드의 수 2개로 변화 전과 후가 동일하게 유지된다.

`w`가 `RED`이기 때문에, `BLACK` children을 가지고, `w`랑 `x.parent`의 색깔을 바꾼 뒤 `x.parent`에서 `leftRotate`를 하면 `w`의 자식 노드였던 아이가 `x`의 새로운 자매노드가 되고 이는 `BLACK` 노드 이므로 Case1 에서 Case 2, 3, 4로 바꿀 수 있다.

***Case 2: x의 자매노드 w가 BLACK이고 w의 두 자식 노드가 BLACK***

![Figure 2. (b)](https://github.com/chatsh1re/chatsh1re.github.io/blob/master/_posts/images/RB%20Tree/case 2.png?raw=true)

*Case 2는 노드 D를 `RED`로 칠하고 `x`가 B를 가리키게 하여 `x`에 의해 나타난 extra black를  트리의 위로 올린다. 만약 Case1을 통하여 Case 2가 진행되면 새로운 노드 `x`가 red-and-black이므로 반복문이 종료될 것이며 그러므로 `c`의 색깔은 RED가 될 것이다.*

> Property 5: `BLACK`이나 `RED` 모두 될 수 있는 `root` 노드 `c`를 고려해야 한다. count(`RED`) = 0 그리고 count(`BLACK`) = 1로 정의한다면, `root`부터 $\alpha$ 까지 `BLACK` 노드의 수는 변환 전 후 모두 2 + count(`c.color`)가 될 것이다. 

`x`와 `w`둘 다 `BLACK`이기 때문에 `x`는 유지하고 `w`만 `RED`로 바꿔주자. 그러면 하나의 `BLACK`이 사라졌기 때문에, 원래 `RED`나 `BLACK`였던 `x.parent`에 extra black를 추가하자. 이 `x.parent`를 새로운 `x`로 두고 `while`문을 반복하자. 만약 Case 1을 통하여 Case 2로 넘어왔다면 원래의 `x.parent`가 `RED`였기 때문에 새로운 `x`는 red-and-black이 될 것이다. 따라서 새로운 `x`의 `color`을 갖고 있는 `c` 역시 `RED`이고 반복문은 끝나게 될 것이다. 그리고 마지막 Line에서 새로운 `x`는 (singly) `BLACK`로 바꿔준다.

***Case 3: x의 자매노드 w가 BLACK이고 w의 left가 RED이며 w의 right는 BLACK인 경우***

![Figure 3. (c)](https://github.com/chatsh1re/chatsh1re.github.io/blob/master/_posts/images/RB%20Tree/case 3.png?raw=true)

*Case 3은 노드 C와 D의 색깔을 바꾸고 `rightRotate`를 하여 Case 4로 바꿔준다.*

> Property 5: 이전과 마찬가지로 count을 활용하여 `c`를 계산한다면, 변환 전과 후의 `BLACK` 노드의 수가 동일하게 유지된다. 

`w`와 `w.left`의 색깔을 바꾸고 `w`에서 `rightRotate`를 하면 `x`의 새로운 자매노드 `w`는 `RED`인 `right` 자식 노드를 가지게 되고 Case 4로 변환된다. 

***Case 4: x의 자매노드 w가 BLACK이고, w의 right가 RED인 경우***

![Figure 4. (d)](https://github.com/chatsh1re/chatsh1re.github.io/blob/master/_posts/images/RB%20Tree/case 4.png?raw=true)

*Case 4는 `x`에 의해 나타난 extra black를 다른 `color`로 바꿔 레드블랙 Properties를 위반하지 않게 한 뒤 `leftRotate`를 하여 반복문을 종료시킨다.*

>Property 5: 이전과 마찬가지로 count을 활용하여 c를 계산한다면, 변환 전과 후의 `BLACK` 노드의 수가 동일하게 유지된다.

약간의 `color` 변환과 `x.parent`에서 `leftRotate`를 통하여 `x`의 extra black를 single black으로 만들어 준다. `x`가 `root`가 된다면 `while`문 조건에 따라 반복문이 종료될 것이다.

## Analysis

`RB-Delete`의 러닝타임을 알아보자. n개의 노드를 가진 레드블랙 트리는 $O(lgn)$ 의 높이를 가지며 `RB-Delete-Fixup`를 재외한 `RB-Delete` 과정은 $O(lgn)$의 시간복잡도를 가진다. `RB-Delete-Fixup` 에서 Case 1, 3, 4는 한번만 실행된 뒤 종료되므로 회전에 걸리는 시간 정도의 시간복잡도를 지니고 Case 2는 실행 될 때마다 `x`가 트리의 위쪽으로 가므로 최대 $O(lgn)$의 시간복잡도를 지닌다. 따라서 `RB-Delete`의 러닝타임은 $O(lgn)$이 된다.

## Code (c)

```c
#include "rbt.h"

//x는 RBT의 성질이 깨진 노드(BLACK), w는x의 자매 노드
void	RBDeleteFixup(t_rbt **root, t_rbt *x, t_rbt *nil)
{
	t_rbt *w;

	while ((x->parent != nil) && (x->color == BLACK))
	{
		if (x == x->parent->left) //x가 부모노드의 left인 경우
		{
			w = x->parent->right;
			if (w->color == RED) //Case 1
			{
				w->color = BLACK;
				w->parent->color = RED;
				leftRotate(&x->parent, nil);
				w = x->parent->right;
			}
			if ((w->left->color == BLACK) && (w->right->color == BLACK)) //Case 2
			{
				w->color = RED;
				x = x->parent;
			}
			else 
			{
				if (w->right->color == BLACK) // Case 3
				{
					w->left->color = BLACK;
					w->color = RED;
					rightRotate(&w, nil);
					w = w->parent->right;
				}
				w->color = x->parent->color; // Case 4
				x->parent->color = BLACK;
				w->right->color = BLACK;
				leftRotate(&x->parent, nil);
				break ;
			}
		}
		else
		{
			w = x->parent->left;
			if (w->color == RED)
			{
				w->color = BLACK;
				w->parent->color = RED;
				rightRotate(&x->parent, nil);
				w = x->parent->left;
			}
			if ((w->right->color == BLACK) && (w->left->color == BLACK))
			{
				w->color = RED;
				x = x->parent;
			}
			else
			{
				if (w->right->color == BLACK)
				{
					w->right->color = BLACK;
					w->color = RED;
					leftRotate(&w, nil);
					w = w->parent->left;
				}
				w->color = x->parent->color;
				x->parent->color = BLACK;
				w->left->color = BLACK;
				rightRotate(&x->parent, nil);
				break ;
			}
		}
	}
}

//u의 위치에 v를 넣어주는 함수
void	RBTransplant(t_rbt **root, t_rbt *nil,  t_rbt *u, t_rbt *v)
{
	if (u->parent == nil)
		*root = v;
	else if (u == u->parent->left)
		u->parent->left = v;
	else 
		u->parent->right = v;
	v->parent = u->parent;
}

t_rbt	*minimum(t_rbt *tree, t_rbt *nil)
{
	while (tree->left != nil)
		tree = tree->left;
	return (tree);
}

//z : 삭제할 노드 y : z자리에 들어올 노드 x : y의 자식노드
void	RBDelete(t_rbt **root, t_rbt *z, t_rbt *nil)
{
	t_rbt *y;
	t_rbt *x;
	int y_original_color;

	y = z;
	y_original_color = y->color;
	if (z->left == nil) //z의 자식 노드가 2개 미만인 경우
	{
		x = z->right;
		RBTransplant(root, nil, z, z->right);
	}
	else if (z->right == nil)
	{
		x = z->left;
		RBTransplant(root, nil, z, z->left);
	}
	else //z의 자식 노드가 2개인 경우
	{
		y = minimum(z->right, nil);
		y_original_color = y->color;
		x = y->right;
		if (y->parent == z) //z의 successor가 바로 왼쪽 자식 노드면 이어주면ok
			x->parent = y;
		else //저 밑에 있는 y도 삭제되므로 그 자리를 고려해줌
		{ 
			RBTransplant(root, nil, y, y->right);
			y->right = z->right;
			y->right->parent = y;
		}
		RBTransplant(root, nil, z, y); //이제 z를 y로 바꿔주면 ok
		y->left = z->left;
		y->left->parent = y;
		y->color = z->color;	
	}
	if (y_original_color == BLACK)
		RBDeleteFixup(root, x, nil);
	free(z);
}

t_rbt	*searchRB(t_rbt **root, t_rbt *nil, int key)
{
	t_rbt *tmp;

	tmp = *root;
	while (tmp != nil)
	{
		if (key == tmp->key)
			return (tmp);
		else if (key < tmp->key)
			tmp = tmp->left;
		else
			tmp = tmp->right;
	}
	return (nil);
}

//삭제할 데이터의 노드를 찾고 삭제한 트리를 리턴
t_rbt	**initRBDeletion(t_rbt **root, t_rbt *nil, int key)
{
	t_rbt *z;

	z = searchRB(root, nil, key);
	if (z == nil)
	{
		printf("해당 노드가 없습니다 \n");
		return (root);
	}
	RBDelete(root, z, nil);
	return (root);
}
```

원본 코드는 [깃허브](https://github.com/chatsh1re/Algorithm/tree/master/tree/Red_black) 에 있으며 make명령어를 통해 컴파일하여 확인해 볼 수 있다.

## 여담

이해하느냐 머리 터질뻔🤯 했다. 도움을 얻고자 검색해 봤으나 c로 구현한 레드블랙 트리 삭제는 거의 나오지 않았고, 이와 다른 방법으로 구현한 사람들도 많았다.

## 참고

*Introduction to Algorithms 3rd edition*