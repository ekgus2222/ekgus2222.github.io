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




> # Box, Gaussian, Median Filter
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
    //필터링
    int filtersize = ui->spin_FilterSize->value();

    //Box Filtering
        //S_n_P noise 이미지 불러오기
    ImageForm* q_pForm_s = 0;

        for(int i=0; i<_plpImageForm->Count();i++)
            if((*_plpImageForm)[i]->ID()=="Salt_and_Pepper Noise")
            {
                q_pForm_s = (*_plpImageForm)[i];
                break;
            }

    KImageColor Salt_Noise_2 = q_pForm_s->ImageColor();

    //Gaussian noise 이미지 불러오기

    ImageForm* q_pForm_g = 0;

        for(int i=0; i<_plpImageForm->Count();i++)
            if((*_plpImageForm)[i]->ID()=="Gaussian Noise")
            {
                q_pForm_g = (*_plpImageForm)[i];
                break;
            }
    KImageColor Gaussian_Noise_2 = q_pForm_g->ImageColor();

    int size = 0.5*filtersize-0.5;
    int box = filtersize*filtersize;
    //qDebug()<<size;
        //Salt Noise
    double sum_r_s,sum_g_s,sum_b_s;
    double sum_r_g,sum_g_g,sum_b_g;

    for(int ii = size; ii<Salt_Noise_2.Row()-size;ii++){
        for(int jj = size; jj<Salt_Noise_2.Col()-size;jj++){
            sum_r_s = 0;
            sum_g_s = 0;
            sum_b_s = 0;

            sum_r_g = 0;
            sum_g_g = 0;
            sum_b_g = 0;
            for(int i = -size; i<size+1; i++){
                for(int j = -size; j<size+1;j++){
                    sum_r_s+=Salt_Noise[ii-i][jj-j].r;
                    sum_g_s+=Salt_Noise[ii-i][jj-j].g;
                    sum_b_s+=Salt_Noise[ii-i][jj-j].b;

                    sum_r_g+=Gaussian_Noise[ii-i][jj-j].r;
                    sum_g_g+=Gaussian_Noise[ii-i][jj-j].g;
                    sum_b_g+=Gaussian_Noise[ii-i][jj-j].b;
                }
            }

            //qDebug()<<sum_r/(filtersize*filtersize);

            Salt_Noise_2[ii][jj].r=sum_r_s/box;
            Salt_Noise_2[ii][jj].g=sum_g_s/box;
            Salt_Noise_2[ii][jj].b=sum_b_s/box;

            Gaussian_Noise_2[ii][jj].r=sum_r_g/box;
            Gaussian_Noise_2[ii][jj].g=sum_g_g/box;
            Gaussian_Noise_2[ii][jj].b=sum_b_g/box;
        }
    }


    //show noise 제거
    ImageForm*  q_pForm3 = new ImageForm(Salt_Noise_2, "Box_Salt_n_Pepper", this);
    _plpImageForm->Add(q_pForm3);
    q_pForm3->show();

    ImageForm*  q_pForm4 = new ImageForm(Gaussian_Noise_2, "Box_Gaussian", this);
    _plpImageForm->Add(q_pForm4);
    q_pForm4->show();
```    




> ## Gaussian Filter

<img src = "https://user-images.githubusercontent.com/69707792/125559377-8c9461e9-089b-4ed2-913d-f2e3d3083ac6.jpg" width = 60%>


- 장점
    - Secondary lobes가 없으므로 정확한 low-pass filter 구현이 가능하다.
    <img src = "https://user-images.githubusercontent.com/69707792/125559797-0f8dc51b-0e50-449d-8bff-514f80cb049b.jpg" width = 60%>


> ### result

![GaussianFilter](https://user-images.githubusercontent.com/69707792/125557750-b2a7be9c-107f-454d-81fd-cdd50a35d687.JPG)

> ### Code
```C++
    //Gaussian 필터링

    //S_n_P noise 이미지 불러오기
    ImageForm* q_pForm_s = 0;

    for(int i=0; i<_plpImageForm->Count();i++)
        if((*_plpImageForm)[i]->ID()=="Salt_and_Pepper Noise")
        {
            q_pForm_s = (*_plpImageForm)[i];
            break;
        }

    KImageColor Salt_Noise_2 = q_pForm_s->ImageColor();

    //Gaussian noise 이미지 불러오기

    ImageForm* q_pForm_g = 0;

    for(int i=0; i<_plpImageForm->Count();i++)
        if((*_plpImageForm)[i]->ID()=="Gaussian Noise")
        {
            q_pForm_g = (*_plpImageForm)[i];
            break;
        }
    KImageColor Gaussian_Noise_2 = q_pForm_g->ImageColor();

    int Gau_filter_size = 8*sigma + 1; //filter size




    int size = std::sqrt(Gau_filter_size);


    //qDebug()<<size;
        //Salt Noise

    double mask_r_s, mask_g_s, mask_b_s;
    double mask_r_g, mask_g_g, mask_b_g;


    for(int ii = size; ii<Salt_Noise_2.Row()-size;ii++){
        for(int jj = size; jj<Salt_Noise_2.Col()-size;jj++){

            mask_r_s = 0;
            mask_g_s = 0;
            mask_b_s = 0;

            mask_r_g = 0;
            mask_g_g = 0;
            mask_b_g = 0;


            for(int i = -size; i<size+1; i++){
                for(int j = -size; j<size+1;j++){
                    mask_r_s += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*Salt_Noise[ii-i][jj-j].r;
                    mask_g_s += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*Salt_Noise[ii-i][jj-j].g;
                    mask_b_s += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*Salt_Noise[ii-i][jj-j].b;

                    mask_r_g += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*Gaussian_Noise[ii-i][jj-j].r;
                    mask_g_g += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*Gaussian_Noise[ii-i][jj-j].g;
                    mask_b_g += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*Gaussian_Noise[ii-i][jj-j].b;
                }
            }

            Salt_Noise_2[ii][jj].r = (1/(2*M_PI*sigma*sigma))*mask_r_s;
            Salt_Noise_2[ii][jj].g = (1/(2*M_PI*sigma*sigma))*mask_g_s;
            Salt_Noise_2[ii][jj].b = (1/(2*M_PI*sigma*sigma))*mask_b_s;

            Gaussian_Noise_2[ii][jj].r = (1/(2*M_PI*sigma*sigma))*mask_r_g;
            Gaussian_Noise_2[ii][jj].g = (1/(2*M_PI*sigma*sigma))*mask_g_g;
            Gaussian_Noise_2[ii][jj].b = (1/(2*M_PI*sigma*sigma))*mask_b_g;



        }
    }


    //show noise 제거
    ImageForm*  q_pForm3 = new ImageForm(Salt_Noise_2, "Gaussian_Salt_n_Pepper", this);
    _plpImageForm->Add(q_pForm3);
    q_pForm3->show();

    ImageForm*  q_pForm4 = new ImageForm(Gaussian_Noise_2, "Gaussian_Gaussian", this);
    _plpImageForm->Add(q_pForm4);
    q_pForm4->show();
```    






> ## Median Filter
- Median Filtering
> ### result

![MedianFilter](https://user-images.githubusercontent.com/69707792/125557741-fda5db35-3fa6-4cdc-9f28-b4ac8881764f.JPG)

> ### Code
```C++
    //필터링
    int filtersize = ui->spin_FilterSize->value();

    //Median Filtering
        //S_n_P noise 이미지 불러오기
    ImageForm* q_pForm_s = 0;

        for(int i=0; i<_plpImageForm->Count();i++)
            if((*_plpImageForm)[i]->ID()=="Salt_and_Pepper Noise")
            {
                q_pForm_s = (*_plpImageForm)[i];
                break;
            }

    KImageColor Salt_Noise_2 = q_pForm_s->ImageColor();

    //Gaussian noise 이미지 불러오기

    ImageForm* q_pForm_g = 0;

        for(int i=0; i<_plpImageForm->Count();i++)
            if((*_plpImageForm)[i]->ID()=="Gaussian Noise")
            {
                q_pForm_g = (*_plpImageForm)[i];
                break;
            }
    KImageColor Gaussian_Noise_2 = q_pForm_g->ImageColor();


    int size = 0.5*filtersize-0.5;
    std::vector<double> v_r_s,v_g_s,v_b_s;
    std::vector<double> v_r_g,v_g_g,v_b_g;
    int box = filtersize*filtersize;

    //qDebug()<<size;
        //Salt Noise


    for(int ii = size; ii<Salt_Noise_2.Row()-size;ii++){
        for(int jj = size; jj<Salt_Noise_2.Col()-size;jj++){

            v_r_s.clear();
            v_g_s.clear();
            v_b_s.clear();

            v_r_g.clear();
            v_g_g.clear();
            v_b_g.clear();


            for(int i = -size; i<size+1; i++){
                for(int j = -size; j<size+1;j++){
                    v_r_s.push_back(Salt_Noise[ii-i][jj-j].r);
                    v_g_s.push_back(Salt_Noise[ii-i][jj-j].g);
                    v_b_s.push_back(Salt_Noise[ii-i][jj-j].b);

                    v_r_g.push_back(Gaussian_Noise[ii-i][jj-j].r);
                    v_g_g.push_back(Gaussian_Noise[ii-i][jj-j].g);
                    v_b_g.push_back(Gaussian_Noise[ii-i][jj-j].b);
                }
            }

            //qDebug()<<sum_r/(filtersize*filtersize);
            sort(v_r_s.begin(),v_r_s.end());
            sort(v_g_s.begin(),v_g_s.end());
            sort(v_b_s.begin(),v_b_s.end());

            sort(v_r_g.begin(),v_r_g.end());
            sort(v_g_g.begin(),v_g_g.end());
            sort(v_b_g.begin(),v_b_g.end());

            Salt_Noise_2[ii][jj].r = v_r_s[box/2];
            Salt_Noise_2[ii][jj].g = v_g_s[box/2];
            Salt_Noise_2[ii][jj].b = v_b_s[box/2];

            Gaussian_Noise_2[ii][jj].r = v_r_g[box/2];
            Gaussian_Noise_2[ii][jj].g = v_g_g[box/2];
            Gaussian_Noise_2[ii][jj].b = v_b_g[box/2];


        }
    }


    //show noise 제거
    ImageForm*  q_pForm3 = new ImageForm(Salt_Noise_2, "Median_Salt_n_Pepper", this);
    _plpImageForm->Add(q_pForm3);
    q_pForm3->show();

    ImageForm*  q_pForm4 = new ImageForm(Gaussian_Noise_2, "Median_Gaussian", this);
    _plpImageForm->Add(q_pForm4);
    q_pForm4->show();
```   
