---
layout: post
title:  "[Coding] SCPC 5회 예선 공굴리기(150)"
subtitle:   "열공코테"
categories: 
    - study
    - coding
tags: [coding]
comments: true
use_math: true
---

# 공 굴리기

> ### 문제 설명
아래 그림에서 보인 것처럼 장애물이 놓인 길을 따라 공을 오른쪽으로 굴릴 때, 공의 중심이 어떤 궤적을 따라 이동하는지 알고자 한다.   
길에 놓인 장애물들은 직사각형으로 표시되고, 모든 장애물은 x 축 상에 놓여있다.   
공이 장애물을 만나 더 이상 움직일 수 없다면 그림에서 보듯이 장애물의 벽 또는 모서리를 따라 넘어가되, 공은 어떤 경우에도 장애물에서 떨어지지 않은 채 장애물의 벽 또는 모서리를 따라 구르며, 아무리 높은 장애물도 넘어 갈 수 있다.    
<br>
공의 반지름, 출발점 위치(즉, 공의 중심의 x 좌표), 도착점 위치, 그리고 모든 장애물의 위치와 크기에 대한 정보가 주어질 때, 공의 중심이 이동한 총 거리를 출력하는 프로그램을 작성하고자 한다.    
<br>
단, 공의 반지름을 R이라 하면, 모든 장애물의 너비는 2R 초과이며, 또한 장애물 사이의 간격도 2R 초과이다. 그리고 출발점과 도착점에서는 공이 항상 바닥에 닿아 있다고 가정하라.   

> ### 풀이


> ### code

```C++
#define _USE_MATH_DEFINES

#include <iostream>
#include <cmath>

using namespace std;

double Answer;
double pi = M_PI;

double cal(double R,double start, double l, double r, double h)
{
	double part1 = 0, part2 = 0;
	double end = 0;
	double angle = 0;
	double interval = 0;

	part1 = (double)((l - start - R) + 2 * (h - R) + (r - l));
	part2 = (double)R * pi;
	end = r + R;
	
	if (h >= R)
	{
		part1 = (double)((l - start - R) + 2 * (h - R) + (r - l));
		part2 = (double)R * pi;
		end = r + R;
	}
	else {
		interval = sqrt(R * R - (R - h) * (R - h));
		//cout << "interval : " << interval << endl;
		end = r + interval;
		
		angle = pi/2 - atan2(R - h, interval);

		part1 = (double)((l - start - interval) + (r - l)); 
		part2 = 2*(double)R * angle;
	}
	
	
	//cout << "part1 : " << part1 << " part2 : " << part2 << " angle : " << angle << endl;
	//cout << fixed;
	//cout.precision(10);
	Answer += part1 + part2;
	
	
	return end;
}

int main(int argc, char** argv)
{
	ios_base::sync_with_stdio(0); cin.tie(0);  cout.tie(0);

	int T, test_case;

	double R = 0, S = 0, E = 0;
	double N = 0, l = 0, r = 0, h = 0;

	double start = 0;

	cin >> T;
	for (test_case = 0; test_case < T; test_case++)
	{
		Answer = 0;

		cin >> R >> S >> E >> N;

		start = S;

		for (int i = 0; i < N; i++)
		{
			cin >> l >> r >> h;

			start = cal(R, start, l, r, h);
		}

		Answer += E - start;


		 // Print the answer to standard output(screen).
		cout << "Case #" << test_case + 1 << endl;
		cout << fixed;
		cout.precision(10);
		cout << Answer << endl;
	}

	return 0;//Your program should return 0 on normal termination.
}

```
