---
layout: post
title:  "[Computer_Vision] 6. Hough Transform"
subtitle:   "정문호 교수님"
categories: 
    - learn
    - computer_vision
tags: [computer_vision]
comments: true
use_math: true
---

# 6. Hough Transform
> # Input Image

![InputImg](https://user-images.githubusercontent.com/69707792/125890718-6d4b6254-f7d2-428f-8eb4-053fcac9b500.JPG)   

* Circle Hough Transform : 반지름이 주어져 있음
* General Hough Transform : 도형의 모양이 좌표로 주어져 있음


> # Hough Transform
- Hough Trnasform이란?
    기본적인 아이디어는 voting이다.
    1. 우선 찾으려는 도형의 파라미터를 변수로 정한다. ( 파라미터를 잘못 정할 경우 옳지 않은 경우가 생길 수 있으므로 모든 경우를 나타낼 수 있는 파라미터를 선정해야 한다.)
    2. 원래의 이미지의 한 점에서 구할 수 있는 모든 경우의 수를 파라미터 공간에 voting한다. 
    3. voting이 가장 많은 파라미터가 우리가 구하려는 도형의 파라미터 변수로 정해진다.

> # Circle Hough Transform

1. circle edge를 찾는 것이기 때문에 edge 점들에서만 voting을 해야한다. 이를 위해 canny edge operator 함수를 이용해 edge를 구한다.
2. 반지름이 103 pixel로 주어져 있으므로 파라미터는 원의 중심점(a,b)이다.
3. edge 점을 지나고 103pixel을 가진 원의 중심을 파라미터 공간에 voting한다.
4. voting이 가장 많은 점을 원의 중심으로 해서 103 pixel의 반지름을 가진 원을 표시한다. 

> ### result

![CircleImg](https://user-images.githubusercontent.com/69707792/125890769-89881e51-4110-4601-8709-95fe6eeb615b.JPG)

