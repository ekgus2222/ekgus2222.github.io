---
layout: post
title:  "[GithubPages] 1.Sepia Tone"
subtitle:   "정문호 교수님"
categories: 
    - learn
    - computer_vision
tags: [computer_vision]
comments: true
use_math: true
---

# 1. Sepia Tone
> ## Color

![color](https://user-images.githubusercontent.com/69707792/124476010-5516ca80-dddd-11eb-9f32-4b8cdaebb01e.jpg)

    
> ## Sepia Tone Transform Algorithm
    1. Transform RGB color to HSV color
    2. Change H and S
    3. Transform HSV color to RGB
    
![sepia_tone_transform](https://user-images.githubusercontent.com/69707792/124476023-58aa5180-dddd-11eb-8aab-7393c9e31312.jpg)



> ## result
- HUE : 110, SAT : 0.3

![sepia_result](https://user-images.githubusercontent.com/69707792/124475758-0b2de480-dddd-11eb-9f2a-d7b97a6934f7.jpg)

![HSV](https://user-images.githubusercontent.com/69707792/124475765-0e28d500-dddd-11eb-95d3-4af6301866d3.jpg)



> ## Code
```C++
void MainFrame::on_buttonSepiaTone_clicked()
{
    KImageColor   icMain;

    //포커스 된 ImageForm으로부터 영상을 가져옴

    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageColor().Address() &&  _q_pFormFocused->ID() == "OPEN")
        icMain = _q_pFormFocused->ImageColor();
    else
        return;

    KImageGray img_Hue(icMain.Row(),icMain.Col()),
               img_Sat(icMain.Row(),icMain.Col()),
               img_Val(icMain.Row(),icMain.Col());

    //hue, saturation 값 가져오기
    double dHue2 = ui->spinHue->value();//text().toDouble();
    double dSat2 = ui->spinSaturation->value(); // 위와 같은 식으로

    //
    //icMain 변환
    //:

    double dMin, dMax, dDiff;
    double dVal;
    double C,X,m;
    double R,G,B;
    double dHue,dSat;

    for(int i=icMain.Row(),ii=0; i; i--,ii++)
        for(int j=icMain.Col(),jj=0; j; j--,jj++)
        {
            dMin  = std::min(std::min(icMain[ii][jj].r,icMain[ii][jj].g),icMain[ii][jj].b);
            dMax  = std::max(std::max(icMain[ii][jj].r,icMain[ii][jj].g),icMain[ii][jj].b);
            dDiff = dMax - dMin;

            //value
            dVal = dMax/255.0;
            img_Val[ii][jj]= dVal*255;

            //saturation
            dSat = dDiff/dMax;

            //hue
            if(dMax == (double)(icMain[ii][jj].r)) //maximun = red
                dHue = 60.0 * (double)(icMain[ii][jj].g-icMain[ii][jj].b)/dDiff;
            else if(dMax == (double)(icMain[ii][jj].g))
                dHue = 60.0 * (double)(icMain[ii][jj].b-icMain[ii][jj].r)/dDiff + 120.0;
            else
                dHue = 60.0 * (double)(icMain[ii][jj].r-icMain[ii][jj].g)/dDiff + 240.0;

            if(dHue == 360.0)
                dHue = 0.0;

            img_Hue[ii][jj]=dHue/360.0 * 255;
            img_Sat[ii][jj]=dSat * 255.0;

            dHue = dHue2;
            dSat = dSat2;

            C = dVal*dSat;
            X = C*(1-std::fabs(fmod(dHue/60,2)-1));
            m = dVal - C;

            if(dHue>=0&&dHue<60)
            {
                R = C;
                G = X;
                B = 0;
            }
            else if (dHue>=60&&dHue<120)
            {
                R = X;
                G = C;
                B = 0;
            }
            else if (dHue>=120&&dHue<180)
            {
                R = 0;
                G = C;
                B = X;
            }
            else if (dHue>=180&&dHue<240)
            {
                R = 0;
                G = X;
                B = C;
            }
            else if (dHue>=240&&dHue<300)
            {
                R = X;
                G = 0;
                B = C;
            }
            else if (dHue>=300&&dHue<360)
            {
                R = C;
                G = 0;
                B = X;
            }


            icMain[ii][jj].r=(R+m)*255.0;
            icMain[ii][jj].g=(G+m)*255.0;
            icMain[ii][jj].b=(B+m)*255.0;

        }

    //출력을 위한 ImageForm 생성
    //sepia
    ImageForm*  q_pForm = new ImageForm(icMain, "Sepia Tone", this);
    _plpImageForm->Add(q_pForm);
    q_pForm->show();


    //HSV
    ImageForm*  q_pHue = new ImageForm(img_Hue,"Hue",this);
    _plpImageForm->Add(q_pHue);
    q_pHue->show();

    ImageForm*  q_pSat = new ImageForm(img_Sat,"Sat",this);
    _plpImageForm->Add(q_pSat);
    q_pSat->show();

    ImageForm*  q_pVal = new ImageForm(img_Val,"Val",this);
    _plpImageForm->Add(q_pVal);
    q_pVal->show();


    //UI 활성화 갱신
    UpdateUI();
}

```
