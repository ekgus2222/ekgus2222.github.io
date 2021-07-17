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


> ### Code
```C++
void MainFrame::on_button_CircleHough_clicked()
{
    KImageGray icMain;

    //포커스 된 ImageForm으로부터 영상을 가져옴
    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageGray().Address() &&  _q_pFormFocused->ID() == "OPEN")
    {
        icMain = _q_pFormFocused->ImageGray();
    }
    else
        return;

    KImageGray icMain2(icMain.Row(),icMain.Col());

    //canny edge operator
    double sigma = ui->SpinBox_sigma->value();

    Canny_Edge(icMain,sigma);

    //Hough Transform
    int** voting = new int*[icMain.Row()]{0,};
    for(int i = 0; i<icMain.Row(); i++)
    {
        voting[i] = new int[icMain.Col()]{0,};
    }





    std::vector<Location> edge;
    Location location;



        //Edge 좌표 받아오기
    for(int ii = 0; ii<icMain.Row();ii++){
        for(int jj = 0; jj<icMain.Col();jj++){

            if(icMain[ii][jj] == 255)
            {
                location.x = ii;
                location.y = jj;
                edge.push_back(location);

                 //qDebug() << "x : " << ii << "y : " << jj;
            }

        }
    }


        //voting
    int x,y;
    int a,b;
    for(int i=0;i<edge.size();i++)
    {
        x = edge[i].x;
        y = edge[i].y;

        //qDebug() << "x : " << x << "y : " << y;

        for(int j=0;j<360;j++){
            a = x-51.5*cos(j*M_PI/180);
            b = y-51.5*sin(j*M_PI/180);


            if(a>0 && a<icMain.Row() && b>0 && b<icMain.Col())
            {
                voting[a][b]++;
                if(icMain2[a][b]+30 < 256)
                    icMain2[a][b] += 30;
                else icMain2[a][b]  = 255;
            }


        }


    }



        //max voting -> 원의 중심점 찾기
    int max = 0;

    for(int ii = 0; ii<icMain.Row();ii++){
        for(int jj = 0; jj<icMain.Col();jj++){
            if(voting[ii][jj]>max){
                a = ii;
                b = jj;
                max = voting[ii][jj];
            }
        }
    }

    for(int ii = 0; ii<icMain.Row();ii++){
        for(int jj = 0; jj<icMain.Col();jj++){
            icMain[ii][jj] = 0;
            //icMain2[ii][jj] = 0;
        }
    }


        //circle
    for(int j=0;j<360;j++)
    {
        x = a-51.5*cos(j*M_PI/180);
        y = b-51.5*sin(j*M_PI/180);
        icMain[x][y] = 255;
    }





    //Show
    ImageForm*  q_pForm1 = new ImageForm(icMain, "Circle Hough Transform", this);
    _plpImageForm->Add(q_pForm1);
    q_pForm1->show();

    ImageForm*  q_pForm2 = new ImageForm(icMain2, "Circle Hough Transform_voting", this);
    _plpImageForm->Add(q_pForm2);
    q_pForm2->show();


}
```




> # General Hough Transform
- 크기와 회전 각도가 고정일 때
    - 파라미터 3개
    - voting 공간 : X_c, Y_c
<img src = "https://user-images.githubusercontent.com/69707792/125898214-428f12e8-cea2-4dfa-a6a6-9d8fec42443d.jpg" width = 80%>

    1. 도형의 중심점을 잡고 파라미터 테이블을 만든다. 이 때 Edge Direction을 key로 사용한다.   
    2. 아래의 수식을 이용해 각 edge에서 table를 돌면서 나올 수 있는 X_c와 Y_c의 경우를 파라미터 공간에 voting한다.
   
<img src = "https://user-images.githubusercontent.com/69707792/125899278-94040ea1-78c3-4afd-b4b8-a1c5bea7d914.jpg" width = 30%>

- 크기와 회전 각도가 바뀔 때
    - 파라미터 5개
    - voting 공간 : X_c, Y_c, s, theta
<img src = "https://user-images.githubusercontent.com/69707792/125898220-2bd669d2-9d45-44cb-bd6b-39a885e76291.jpg" width = 80%>
    
    1. 크기와 회전 각도가 고정일 때와 같은 파라미터 테이블을 만든다.
    2. 위의 그림과 같이 voting 한다.
    
  
> ### result

![GeneralImg](https://user-images.githubusercontent.com/69707792/125890741-bd7305d4-6d4e-4409-a2a2-4dcc0b1405ff.JPG)


