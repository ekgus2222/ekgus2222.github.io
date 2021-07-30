---
layout: post
title:  "[Computer_Vision] 8. Optical Flow"
subtitle:   "정문호 교수님"
categories: 
    - learn
    - computer_vision
tags: [computer_vision]
comments: true
use_math: true
---

# 8. Optical Flow
> # Input Image

![inputImg](https://user-images.githubusercontent.com/69707792/126754620-88c9a40c-014e-4ec7-b9c3-e761f342c393.JPG)

![InputData](https://user-images.githubusercontent.com/69707792/126754623-9f0ece09-170a-4299-946c-e481bc8410c1.JPG)


> # Optical Flow
- Optical Flow란?   
    영상 내 물체의 움직임 패턴

- 가정   
    1. 연속된 프레임 사이에서 움직이는 물체의 픽셀 intensity는 변함이 없다. (color constancy)
    2. 이웃하는 픽셀은 비슷한 움직임을 가진다.(Brightness Constraint)

- Aperture Problem   
    하나의 픽셀만 관찰할 경우 조건 부족으로 인해 실제 움직임을 잘못 판별하는 문제가 발생함.

- Lucas & Kanade Method   
    주변 픽셀은 유사한 움직임을 갖는다는 조건을 이용해 추가적인 equation을 구함으로써 해결함.
    
    
> ### result

![1to2](https://user-images.githubusercontent.com/69707792/126754645-02da2e99-ff43-44af-a9ef-c6d3b73a8661.JPG)

![2to3](https://user-images.githubusercontent.com/69707792/126754648-65a2916f-b332-4aee-a370-19ad4855a7aa.JPG)

![3to4](https://user-images.githubusercontent.com/69707792/126754651-ca8093eb-6a31-4fae-9b3f-1862cea37fd8.JPG)


- 범위를 1~4로 설정했을 때의 결과이다.   
- 정지해 있는 산은 변화가 적고 상대적으로 움직이거나 밝기 변화가 있는 폭포와 하늘은 변화가 큰 모습을 볼 수 있다.

> ### Code
```C++
void MainFrame::on_button_opticalflow_clicked()
{
    int StartImgnum = ui->spinBox_StartImg->value();
    int EndImgnum = ui->spinBox_EndImg->value();

    std::vector<KImageGray> Imgvec;

    QString q_fileName;

    for(int i = StartImgnum; i <= EndImgnum; i++)
    {
        if(i<10)
        {
            q_fileName = QString::fromStdString("./data/yos.0" + std::to_string(i) + ".pgm");
        }
        else q_fileName = QString::fromStdString("./data/yos." + std::to_string(i) + ".pgm");

        ImageForm*  q_pForm = new ImageForm(q_fileName, "Open", this);
        _plpImageForm->Add(q_pForm);

        KImageGray* PGM = &q_pForm->ImageGray();//.GaussianSmoothed(2);
        Imgvec.emplace_back(*PGM);
    }

    qDebug()<<"1";

    int row = Imgvec[0].Row();
    int col = Imgvec[0].Col();

    UV** mat_uv = new UV*[row]{0,};

    for(int i = 0; i < row; i++)
    {
        mat_uv[i] = new UV[col]{ {0,0},};
    }


    //Optical_Flow(Imgvec[0],Imgvec[2],mat_uv);
    //Draw(Imgvec[0],mat_uv);


    for(int i = 0; i < Imgvec.size()-2; i++)
    {
        Optical_Flow(Imgvec[i],Imgvec[i+2],mat_uv);
        Draw(Imgvec[i],mat_uv);

        ImageForm*  q_pForm2 = new ImageForm(Imgvec[i],"Optical Flow", this);
        _plpImageForm->Add(q_pForm2);
        q_pForm2->show();
    }




}
```


```C++
typedef struct UV{
    double u;
    double v;
}UV;
```
