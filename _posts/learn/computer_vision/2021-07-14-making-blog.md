---
layout: post
title:  "[Computer_Vision] 3. Histogram Equalization & Histogram Matching"
subtitle:   "정문호 교수님"
categories: 
    - learn
    - computer_vision
tags: [computer_vision]
comments: true
use_math: true
---

# 4. Gaussian noise and salt&pepper noise & Box, Gaussian, Median Filter

> # Noise
> ## Input Image

![InputImg](https://user-images.githubusercontent.com/69707792/125555395-7eaab96f-eac3-4383-9644-032b8e893460.JPG)

> ## Gaussian Noise
- Gaussian Noise란?
    - 정규분포 Noise, 각 픽셀과 관련없이 독립적으로 생김.
    - 아래 수식에서 n은 정규분포를 가지는 잡음   <br>
    <img src = "https://user-images.githubusercontent.com/69707792/125557141-46957d2a-7487-47f6-a7e5-0346f1f504b6.jpg" width = 40%>


> ## Salt&Pepper Noise
- Salt&Pepper Noise란?
    - impulsive noise or spike noise
    - 0 또는 1이 랜덤하게 생김 <br>
    <img src = "https://user-images.githubusercontent.com/69707792/125557473-3849fc4b-3658-49e5-ab15-0ef5e0a33073.jpg" width = 60%>

> ### result

![noise](https://user-images.githubusercontent.com/69707792/125555564-74987786-a87b-439a-bbf5-3bbf478077f7.JPG)


> ### Code
```C++
//포커스 된 ImageForm으로부터 영상을 가져옴
    KImageColor Gaussian_Noise;
    KImageColor Salt_Noise;

    //포커스 된 ImageForm으로부터 영상을 가져옴
    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageColor().Address() &&  _q_pFormFocused->ID() == "OPEN")
    {
        Gaussian_Noise = _q_pFormFocused->ImageColor();
        Salt_Noise = _q_pFormFocused->ImageColor();
    }
    else
        return;




    //Gaussian Noise 생성
    double sigma = ui->spin_GaussianParameter->value();

    KGaussian GauN;
    GauN.Create(0, sigma*sigma);
    GauN.OnRandom(Gaussian_Noise.Size());

    double dNoise = 0;
    double Kp = 8;
    for(int i = 0; i<Gaussian_Noise.Row();i++){
        for(int j = 0; j<Gaussian_Noise.Col();j++){
            dNoise = GauN.Generate();

            if(Gaussian_Noise[i][j].r + dNoise*Kp > 255)
                Gaussian_Noise[i][j].r = 255;
            else if(Gaussian_Noise[i][j].r +dNoise*Kp < 0)
                Gaussian_Noise[i][j].r = 0;
            else
                Gaussian_Noise[i][j].r += dNoise*Kp;

            if(Gaussian_Noise[i][j].g + dNoise*Kp > 255)
                Gaussian_Noise[i][j].g = 255;
            else if(Gaussian_Noise[i][j].g +dNoise*Kp < 0)
                Gaussian_Noise[i][j].g = 0;
            else
                Gaussian_Noise[i][j].g += dNoise*Kp;

            if(Gaussian_Noise[i][j].b + dNoise*Kp > 255)
                Gaussian_Noise[i][j].b = 255;
            else if(Gaussian_Noise[i][j].b +dNoise*Kp < 0)
                Gaussian_Noise[i][j].b = 0;
            else
                Gaussian_Noise[i][j].b += dNoise*Kp;
        }
    }


    //Salt Noise 생성
    srand((int)time(NULL));


    double Thres = 0.005;
    double minus_Thres = (double)(1-Thres);
    double random;


    for(int i=0; i<Salt_Noise.Row(); i++){
        for(int j=0;j<Salt_Noise.Col();j++){

            random = (double) rand()/RAND_MAX;

            if(random<Thres)
            {
                Salt_Noise[i][j].r = 0;
                Salt_Noise[i][j].g = 0;
                Salt_Noise[i][j].b = 0;
            }
            else if(random>minus_Thres){
                Salt_Noise[i][j].r = 255;
                Salt_Noise[i][j].g = 255;
                Salt_Noise[i][j].b = 255;
            }


        }
    }



    //show noise
    ImageForm*  q_pForm1 = new ImageForm(Salt_Noise, "Salt_and_Pepper Noise", this);
    _plpImageForm->Add(q_pForm1);
    q_pForm1->show();

    ImageForm*  q_pForm2 = new ImageForm(Gaussian_Noise, "Gaussian Noise", this);
    _plpImageForm->Add(q_pForm2);
    q_pForm2->show();
```




> # Box, Median, Gaussian Filter
> ## Input Image

![noise](https://user-images.githubusercontent.com/69707792/125557647-4c085b6a-c7d3-485a-abe1-be0e57939021.JPG)



> ## Box Filter
- Box Filtering 
    - 주변 픽셀값의 평균을 얻는 방식의 이미지 필터링
    - 이미지 샘플과 filter kernel을 곱해서 필터링된 결과값을 얻음
    <img src = "https://user-images.githubusercontent.com/69707792/125558328-7f216a54-b9d2-4134-b6e7-91a6703d4054.jpg" width = 40%>
    
- 장점
    - 빠르다
    - Box(i+1) = Box(i) - neighborhood(i)[first] + neighborhood(i+1)[last]

- 단점
    - Signal frequencies shared with noise are lose
    - Impulse noise is diffused but not removed
    - The secondary lobes let noise into the filtered image
    ![sinc](https://user-images.githubusercontent.com/69707792/125558997-379d0e7a-ea18-4d0f-a6cd-e92f82dff2d0.jpg)


> ### result

![BoxFilter](https://user-images.githubusercontent.com/69707792/125557704-b363897d-2f3d-4281-bac0-8e3d95bc327d.JPG)

> ### Code
```C++

```    




> ## Median Filter

> ### result

![MedianFilter](https://user-images.githubusercontent.com/69707792/125557741-fda5db35-3fa6-4cdc-9f28-b4ac8881764f.JPG)

> ### Code
```C++

```    




> ## Gaussian Filter

> ### result

![GaussianFilter](https://user-images.githubusercontent.com/69707792/125557750-b2a7be9c-107f-454d-81fd-cdd50a35d687.JPG)

> ### Code
```C++

```    


