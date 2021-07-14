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
    - 각 픽셀과 관련없이 독립적으로 생김.
    - $\hat I(i,j) = I(i,j) + n(i,j) $
> ## Salt&Pepper Noise
- Salt&Pepper Noise란?

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




> # Histogram Matching
> ## Input Image

![imgInput](https://user-images.githubusercontent.com/69707792/125427161-b15d05f5-a5c2-46cc-b9f9-8d2d4545e7a8.JPG)

> ## Histogram Matching
- Histogram Matching이란?
    특정한 histogram과 Image의 histogram을 matching 시키는 것

- idea

![idea](https://user-images.githubusercontent.com/69707792/125431342-4763a28d-02e1-4a01-b58c-2c9b4807bf7a.jpg)

> ### result

![resultHMA](https://user-images.githubusercontent.com/69707792/125427256-65065fa4-f1b7-4a21-934d-0732eadfd720.JPG)


> ### Code
```C++
void MainFrame::on_buttonHMA_clicked()
{
    //포커스 된 ImageForm으로부터 영상을 가져옴
    KImageColor Source;

    //포커스 된 ImageForm으로부터 영상을 가져옴
    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageColor().Address() &&  _q_pFormFocused->ID() == "OPEN")
    {
        Source = _q_pFormFocused->ImageColor();
    }
    else
        return;


    ImageForm* q_pForm = 0;

        for(int i=0; i<_plpImageForm->Count();i++)
            if((*_plpImageForm)[i]->ID()=="target")
            {
                q_pForm = (*_plpImageForm)[i];
                break;
            }

    KImageColor Target=q_pForm->ImageColor();



    //get histogram
    int histo_S_R[256] = {0, };
    int histo_S_G[256] = {0, };
    int histo_S_B[256] = {0, };

    int histo_T_R[256] = {0, };
    int histo_T_G[256] = {0, };
    int histo_T_B[256] = {0, };

    for(unsigned int i=0; i<Source.Row(); i++){
        for(unsigned int j=0; j<Source.Col(); j++)
        {
            histo_S_R[Source[i][j].r] += 1;
            histo_S_G[Source[i][j].g] += 1;
            histo_S_B[Source[i][j].b] += 1;
        }
    }

    for(unsigned int i=0; i<Target.Row(); i++){
        for(unsigned int j=0; j<Target.Col(); j++)
        {
            histo_T_R[Target[i][j].r] += 1;
            histo_T_G[Target[i][j].g] += 1;
            histo_T_B[Target[i][j].b] += 1;
        }
    }


    //get P
    double P_S_R[256] = {0, };
    double P_S_G[256] = {0, };
    double P_S_B[256] = {0, };

    double P_T_R[256] = {0, };
    double P_T_G[256] = {0, };
    double P_T_B[256] = {0, };


    for(unsigned int t=0; t<256; t++){

        P_S_R[t] = (double)histo_S_R[t]/(double)Source.Size();
        P_S_G[t] = (double)histo_S_G[t]/(double)Source.Size();
        P_S_B[t] = (double)histo_S_B[t]/(double)Source.Size();

        P_T_R[t] = (double)histo_T_R[t]/(double)Target.Size();
        P_T_G[t] = (double)histo_T_G[t]/(double)Target.Size();
        P_T_B[t] = (double)histo_T_B[t]/(double)Target.Size();

        //qDebug() << t << " : " << P_S_R[t];

    }


    //get y , yp
    double y_R[256] = {0, };
    double y_G[256] = {0, };
    double y_B[256] = {0, };

    double yp_R[256] = {0, };
    double yp_G[256] = {0, };
    double yp_B[256] = {0, };

    y_R[0] = P_S_R[0];
    y_G[0] = P_S_G[0];
    y_B[0] = P_S_B[0];

    yp_R[0] = P_T_R[0];
    yp_G[0] = P_T_G[0];
    yp_B[0] = P_T_B[0];

    for(unsigned int r=1; r<256; r++){
        y_R[r] = y_R[r-1] + P_S_R[r];
        //r_R[r] = T_R[r] * 255;
        y_G[r] = y_G[r-1] + P_S_G[r];
        //r_G[r] = T_G[r] * 255;
        y_B[r] = y_B[r-1] + P_S_B[r];
        //r_B[r] = T_B[r] * 255;

        yp_R[r] = yp_R[r-1] + P_T_R[r];
        //r_R[r] = T_R[r] * 255;
        yp_G[r] = yp_G[r-1] + P_T_G[r];
        //r_G[r] = T_G[r] * 255;
        yp_B[r] = yp_B[r-1] + P_T_B[r];
        //r_B[r] = T_B[r] * 255;

        //qDebug() << r << " : " << yp_R[r];

    }


    int tr_R[256] = {0, };
    int tr_G[256] = {0, };
    int tr_B[256] = {0, };

    for(unsigned int i=0; i<256; i++){
         double min_R = 100000.0, min_G = 100000.0, min_B = 100000.0;
        for(unsigned int j = 0; j<256; j++)
        {
            if(min_R>std::fabs(y_R[i]-yp_R[j]))
            {
                min_R = std::fabs(y_R[i]-yp_R[j]);
                tr_R[i] = j;
            }

            if(min_G>std::fabs(y_G[i]-yp_G[j]))
            {
                min_G = std::fabs(y_G[i]-yp_G[j]);
                tr_G[i] = j;
            }

            if(min_B>std::fabs(y_B[i]-yp_B[j]))
            {
                min_B = std::fabs(y_B[i]-yp_B[j]);
                tr_B[i] = j;
            }
        }
        //qDebug() << i << " : " << min_R;
    }


    for(unsigned int i=0; i<256; i++)
    {
        qDebug() << i << " : " << tr_R[i];
    }


    for(unsigned int i=0; i<Source.Row(); i++){
        for(unsigned int j=0; j<Source.Col(); j++)
        {
            Source[i][j].r=tr_R[Source[i][j].r];
            Source[i][j].g=tr_G[Source[i][j].g];
            Source[i][j].b=tr_B[Source[i][j].b];
        }
    }



    ImageForm*  q_pForm2 = new ImageForm(Source, "Histogram Matching", this);
    _plpImageForm->Add(q_pForm2);
    q_pForm2->show();

    //ImageForm*  q_pForm1 = new ImageForm(Target, "target", this);
    //_plpImageForm->Add(q_pForm1);
    //q_pForm1->show();

}
```    


