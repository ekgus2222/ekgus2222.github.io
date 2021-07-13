![histo](https://user-images.githubusercontent.com/69707792/125429033-9e871a6b-f5c6-48a6-991a-c72d0f63447f.jpg)
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

# 3. Histogram Equalization & Histogram Matching

> # Histogram Equalization
> ## Input Image

![inputImg](https://user-images.githubusercontent.com/69707792/125424580-e29eaa8d-3291-4c47-a8ef-aecbe75f5c07.JPG)

> ## Histogram Equalization
- Histogram Equalization이란?
    이미지의 밝기를 조정해 contrast를 개선한다.   
    contrast 높다 = 분산이 크다 = 분산이 가장 크려면 밝기 분포가 일정해야 한다 = 선명하다
    
![histo](https://user-images.githubusercontent.com/69707792/125429047-f6b3af06-98f9-4fe2-b223-17bac9261da5.jpg)

   
> ### result

![resultHEQ](https://user-images.githubusercontent.com/69707792/125427240-93288cdf-8f33-4f9b-b889-41fb4a4a4406.JPG)


> ### Code
```C++
void MainFrame::on_buttonHEQ_clicked()
{
    KImageColor icMain;

    //포커스 된 ImageForm으로부터 영상을 가져옴
    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageColor().Address() &&  _q_pFormFocused->ID() == "OPEN")
    {
        icMain = _q_pFormFocused->ImageColor();
    }
    else
        return;


    //To get its histogramming
    int histo_R[256] = {0, };
    int histo_G[256] = {0, };
    int histo_B[256] = {0, };

    double P_R[256] = {0, };
    double P_G[256] = {0, };
    double P_B[256] = {0, };

    double T_R[256] = {0, };
    double T_G[256] = {0, };
    double T_B[256] = {0, };

    double r_R[256] = {0, };
    double r_G[256] = {0, };
    double r_B[256] = {0, };


    //histogram
    for(unsigned int i=0; i<icMain.Row(); i++){
        for(unsigned int j=0; j<icMain.Col(); j++)
        {
            histo_R[icMain[i][j].r] += 1;
            histo_G[icMain[i][j].g] += 1;
            histo_B[icMain[i][j].b] += 1;

        }
    }

    for(unsigned int t=0; t<256; t++){

        P_R[t] = (double)histo_R[t]/(double)icMain.Size();
        P_G[t] = (double)histo_G[t]/(double)icMain.Size();
        P_B[t] = (double)histo_B[t]/(double)icMain.Size();

        //qDebug() << t << " : " << P_R[t];

    }

    T_R[0] = P_R[0];
    T_G[0] = P_G[0];
    T_B[0] = P_B[0];

    // 1/255 * r = T_R[r]

    for(unsigned int r=1; r<256; r++){
        T_R[r] = T_R[r-1] + P_R[r];
        r_R[r] = T_R[r] * 255;
        T_B[r] = T_G[r-1] + P_G[r];
        r_G[r] = T_G[r] * 255;
        T_G[r] = T_B[r-1] + P_B[r];
        r_B[r] = T_B[r] * 255;

        //qDebug() << r << " : " << T_R[r];

    }


    for(unsigned int i=0; i<icMain.Row(); i++){
        for(unsigned int j=0; j<icMain.Col(); j++)
        {
            icMain[i][j].r = r_R[icMain[i][j].r];
            icMain[i][j].g = r_R[icMain[i][j].g];
            icMain[i][j].b = r_R[icMain[i][j].b];
        }
    }

    ImageForm*  q_pForm2 = new ImageForm(icMain, "HEQ", this);
    _plpImageForm->Add(q_pForm2);
    q_pForm2->show();
}
```




> # Histogram Matching
> ## Input Image

![imgInput](https://user-images.githubusercontent.com/69707792/125427161-b15d05f5-a5c2-46cc-b9f9-8d2d4545e7a8.JPG)

> ## Histogram Matching
   



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


