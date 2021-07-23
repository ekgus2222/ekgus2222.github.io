---
layout: post
title:  "[Computer_Vision] 7. SIFT"
subtitle:   "정문호 교수님"
categories: 
    - learn
    - computer_vision
tags: [computer_vision]
comments: true
use_math: true
---

# 7. SIFT
> # Input Image

![InputImg](https://user-images.githubusercontent.com/69707792/126583418-c84d35fd-0846-4f36-bbe0-1e9b595def5c.JPG) 

> # SIFT
- SIFT란?
    이미지의 크기와 회전에 불변하는 특징을 추출하는 알고리즘

> ## Scale-Space

![Scale-Space](https://user-images.githubusercontent.com/69707792/126583632-4305cade-dd23-4caf-8f28-b433fc022f3b.jpg)

sigma : 1

> ## DOG

![DOG](https://user-images.githubusercontent.com/69707792/126583651-4f89800d-c1a5-45af-a786-ab16f49b9720.jpg)

> ### result

![SIFT](https://user-images.githubusercontent.com/69707792/126583658-ae947822-372a-48f3-8afc-caeb0863e9e3.jpg)

SIFT 특징을 찾고 DB 생성

> ### Code
```C++
void MainFrame::on_button_GSS_DOG_clicked()
{
    KImageGray icMain,Origin,Half1,Half2;



    //포커스 된 ImageForm으로부터 영상을 가져옴
    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageGray().Address() &&  _q_pFormFocused->ID() == "OPEN")
    {
        icMain = _q_pFormFocused->ImageGray();
    }

    else
        return;

    KImageColor icMain2(icMain.GrayToRGB()), icMain3(icMain.GrayToRGB());

    //Scale space 생성
    double sigma = ui->Spinsigma->value();

    KImageDouble igMain(icMain);
    Origin = igMain.ToGray();
    Half1 = igMain.HalfSize().ToGray();
    Half2 = igMain.HalfSize().HalfSize().ToGray();

    std::vector<std::vector<KImageDouble>> Octave(3); //Octave1,2,3


    for(int j=0;j<5;j++)
    {
        //qDebug() <<"1";
        Octave[0].push_back(Gaussian_Filter(Origin,sigma*pow(1.41,(j-1))));
        //qDebug() <<"2";
        Octave[1].push_back(Gaussian_Filter(Half1,sigma*pow(1.41,(j-1))));
        //qDebug() <<"3";
        Octave[2].push_back(Gaussian_Filter(Half2,sigma*pow(1.41,(j-1))));

    }



    //Dog
    std::vector<std::vector<KImageDouble>> Dog(3);
    for(int i = 0; i < 3;i++)
    {
        KImageDouble Dog_img(Octave[i][0].Row(),Octave[i][0].Col());

        for(int octave=1;octave<5;octave++)
        {

            for(int ii=0;ii<Octave[i][0].Row();ii++)
            {
                for(int jj=0;jj<Octave[i][0].Col();jj++)
                {
                    Dog_img[ii][jj] = Octave[i][octave][ii][jj]-Octave[i][octave-1][ii][jj];
                }
            }

            Dog[i].push_back(Dog_img);
            //qDebug() <<Octave[i][0].Row();
        }
    }



    //KeyPoint & filtering
    std::vector<std::vector<real_key>> KeyPoint = Find_Key_Point(Dog); //루트2 시그마, 2시그마

    qDebug() << KeyPoint[0].size();
    qDebug() << KeyPoint[1].size();

    int x, y;


    


    Orientation(KeyPoint,Octave[0],sigma); //orientation
    KeyPoint_descriptor(KeyPoint,Octave[0],sigma);


    for(int scale = 0; scale<KeyPoint.size(); scale++) //root 2 sigma, 2 sigma
    {
        for(int i=0;i<KeyPoint[scale].size();i++)
        {
            Mark_KeyPoint(icMain, KeyPoint[scale][i].magnitude, KeyPoint[scale][i].direction, KeyPoint[scale][i].x, KeyPoint[scale][i].y);
        }
    }

    std::ofstream writeFile("sift.txt");


    for(int i = 0; i<KeyPoint.size();i++)
    {
        for(int j=0;j<KeyPoint[i].size();j++)
        {
            //KeyPoint[i][j].x " " KeyPoint[i][j].y " " KeyPoint[i][j].direction " " KeyPoint[i][j].magnitude
            //"\n"
            writeFile << KeyPoint[i][j].x << " " << KeyPoint[i][j].y << " " << KeyPoint[i][j].direction << " " << KeyPoint[i][j].magnitude
                      << "\n";


            for(int f = 0;f<4;f++)
            {
                for(int r = 0; r<KeyPoint[i][j].feature[f].size();r++)
                //KeyPoint[i][j].feature[f][r] " "
                    writeFile << KeyPoint[i][j].feature[f][r] << " ";
                writeFile << "\n";

            }
            //"\n"
            writeFile << "\n";
        }
      }




    //show

    ImageForm*  q_pForm1 = new ImageForm(icMain, "KeyPoint", this);
    _plpImageForm->Add(q_pForm1);
    q_pForm1->show();



    ImageForm*  q_pForm2 = new ImageForm(Show(Octave), "Scale-Space", this);
    _plpImageForm->Add(q_pForm2);
    q_pForm2->show();

    ImageForm*  q_pForm3 = new ImageForm(Show(Dog), "Dog", this);
    _plpImageForm->Add(q_pForm3);
    q_pForm3->show();

    

}
```

```C++
KImageDouble Gaussian_Filter(KImageGray& icMain, double sigma)
{

    KImageDouble Return(icMain);

    int row = icMain.Row();
    int col = icMain.Col();

    int Gau_filter_size = 8*sigma + 1; //filter size

    int size = std::sqrt(Gau_filter_size);

    //qDebug()<<size;

    double mask;



    for(int ii = 0; ii<row;ii++)
    {
        for(int jj = 0; jj<col;jj++)
        {

            if(ii<size || ii>=row-size || jj<size || jj>=col-size)
            {
                Return[ii][jj] = 0;
                continue;
            }
            mask = 0;


            for(int i = -size; i<size+1; i++)
            {
                for(int j = -size; j<size+1;j++)
                {
                    mask += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*icMain[ii-i][jj-j];

                }
            }

            Return[ii][jj] = (1/(2*M_PI*sigma*sigma))*mask;
        }

    }

    //qDebug()<<"good";
    return Return;
}
```



```C++
KImageGray Show(std::vector<std::vector<KImageDouble>>& Octave){

    int octave_que_size = Octave.size();
    int merged_row = Octave.front().front().Row() * 3;
    int merged_col = Octave.front().front().Col() * Octave.front().size();

    if (merged_col >= 1920) {
        merged_col = 1920;
    }
    KImageGray Base(merged_row, merged_col);


    std::vector<KImageDouble> img_vec;
    KImageGray now;

    int sum_of_prev_row = 0;


    for(int octave = 0; octave<Octave.size();octave++)
    {

        img_vec = Octave[octave];

        int img_vec_size = img_vec.size(); //6
        int each_row = img_vec.front().Row(); //359
        int each_col = img_vec.front().Col(); //640
        int poor_column = 0;


        for(int img = 0; img<img_vec_size;img++)
        {

            //qDebug()<<"ok";
            now = img_vec[img].ToGray();

            for(int i=0;i<each_row;i++)
            {
                for(int j =0; j<each_col;j++)
                {
                    Base[i+sum_of_prev_row][j+(img-poor_column)*each_col] = now[i][j];
                }
            }

            // 다음에 이어 붙일 이미지가 존재하지만 모니터의 가로 길이가 부족할 때 한 칸 아래로 내림
            if (each_col - 1 + ((img - poor_column) + 1) * each_col > merged_col && img + 1 < img_vec_size)
            {
                  sum_of_prev_row += each_row;
                  poor_column = img + 1;
            }
            //qDebug()<<"end";

        }
        sum_of_prev_row += each_row;

    }


    return Base;
}
```




```C++
bool IsPeak(std::vector<KImageDouble>& Dog_Scale,int i,int ii, int jj)
{
    if(Dog_Scale[i][ii][jj]<0.03) return false;

    if(Dog_Scale[i][ii][jj]>0) //극대
    {
         if(Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii-1][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii-1][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii-1][jj+1]||
            Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii][jj+1]||
            Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii+1][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii+1][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i-1][ii+1][jj+1]||


            Dog_Scale[i][ii][jj]<Dog_Scale[i][ii-1][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i][ii-1][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i][ii-1][jj+1]||
            Dog_Scale[i][ii][jj]<Dog_Scale[i][ii][jj-1]||                                               Dog_Scale[i][ii][jj]<Dog_Scale[i][ii][jj+1]||
            Dog_Scale[i][ii][jj]<Dog_Scale[i][ii+1][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i][ii+1][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i][ii+1][jj+1]||

            Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii-1][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii-1][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii-1][jj+1]||
            Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i][ii+1][jj+1]||
            Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii+1][jj-1]||Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii+1][jj]||Dog_Scale[i][ii][jj]<Dog_Scale[i+1][ii+1][jj+1]) return false;

     }
     else
     {
         if(Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii-1][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii-1][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii-1][jj+1]||
            Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii][jj+1]||
            Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii+1][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii+1][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i-1][ii+1][jj+1]||


            Dog_Scale[i][ii][jj]>Dog_Scale[i][ii-1][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i][ii-1][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i][ii-1][jj+1]||
            Dog_Scale[i][ii][jj]>Dog_Scale[i][ii][jj-1]||                                               Dog_Scale[i][ii][jj]>Dog_Scale[i][ii][jj+1]||
            Dog_Scale[i][ii][jj]>Dog_Scale[i][ii+1][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i][ii+1][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i][ii+1][jj+1]||

            Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii-1][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii-1][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii-1][jj+1]||
            Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i][ii+1][jj+1]||
            Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii+1][jj-1]||Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii+1][jj]||Dog_Scale[i][ii][jj]>Dog_Scale[i+1][ii+1][jj+1]) return false;
      }

    return true;

}
```




```C++
std::vector<std::vector<real_key>> Find_Key_Point(std::vector<std::vector<KImageDouble>>& Dog)
{

    double thres = 1.5;
    double r = thres;
    double b = pow(r+1,2)/r;
    double Dxx,Dyy,Dxy;
    double TrH, DetH;
    double a;

    real_key key;
    std::vector<std::vector<real_key>> key_vec(2);

    //scale = 0;
    for(int dog = 0; dog<3; dog++){ //octave 별 dog 선택
        for(int i=1;i<4;i++) //scale 변화
        {
            for(int ii=1;ii<Dog[dog][i].Row()-1;ii++)
            {
                for(int jj=1;jj<Dog[dog][i].Col()-1;jj++)
                {
                    //qDebug() << dog <<" "<<ii<<jj;


                    if(IsPeak(Dog[dog],i,ii,jj)) //6개 이미지 넘겨줌
                    {


                        Dxx = Dog[dog][i][ii][jj+1]+Dog[dog][i][ii][jj-1]-2*Dog[dog][i][ii][jj];
                        Dyy = Dog[dog][i][ii+1][jj]+Dog[dog][i][ii-1][jj]-2*Dog[dog][i][ii][jj];
                        Dxy = ((Dog[dog][i][ii+1][jj+1]-Dog[dog][i][ii+1][jj-1]) - (Dog[dog][i][ii-1][jj+1] - Dog[dog][i][ii-1][jj-1]))/4.0;
                        TrH = Dxx + Dyy;
                        DetH = Dxx*Dyy - Dxy*Dxy;

                        a = TrH*TrH/DetH;

                        //qDebug() << Dxx<<" "<<Dyy<<" "<<" " <<Dxy<<" " << DetH;
                        if(DetH<=0) continue;
                        //if(a < b) KeyPoint[ii][jj] = 255;
                        if(a >= b) continue;

                        //KeyPoint
                        key.x = ii*pow(2,dog);
                        key.y = jj*pow(2,dog);
                        key.scale = i;


                        key_vec[i-1].push_back(key);

                    }

                }

            }

        }
    }

    return key_vec;
}
```



```C++
void Orientation(std::vector<std::vector<real_key>>& KeyPoint, std::vector<KImageDouble> Octave_0,double sigma)
{

    KImageDouble Scale_img;
    int x,y;
    double mag;
    double phase;
    double gau_kenel;

    double bucket[36] = {0,};

    int x_t,y_t;

    int index;

    double max = 0.;
    int dir;

    //int cnt = 1;

    int KeyPoint_scale_size = 0;

    for(int scale = 0; scale<KeyPoint.size(); scale++) //root 2 sigma, 2 sigma
    {
        Scale_img = Octave_0[scale+1]; //origin roo2, origin 2
        sigma = sigma*pow(1.4,scale+1);
        KeyPoint_scale_size = KeyPoint[scale].size();
        for(int i=0;i<KeyPoint_scale_size;i++)
        {
            x = KeyPoint[scale][i].x;
            y = KeyPoint[scale][i].y;

            //16개 좌표 x+w_x, y+w_y
            for(int w_x = -5; w_x < 6; w_x++)
            {
                for(int w_y = -5; w_y < 6; w_y++)
                {
                    x_t = x+w_x;
                    y_t = y+w_y;

                    if(x_t>Scale_img.Row()-2||x_t<1||y_t>Scale_img.Col()-2||y_t<1) continue;
                    gau_kenel = 1./(2*M_PI*sigma*sigma) * exp(-0.5*(w_x*w_x + w_y*w_y)/(sigma*sigma));

                    mag = gau_kenel * sqrt(pow(Scale_img[x_t+1][y_t]-Scale_img[x_t-1][y_t],2)
                                       +pow(Scale_img[x_t][y_t+1]-Scale_img[x_t][y_t-1],2));

                    phase = atan2(Scale_img[x_t][y_t+1]-Scale_img[x_t][y_t-1],
                                  Scale_img[x_t+1][y_t]-Scale_img[x_t-1][y_t])*180/M_PI+180.0;

                    //if(phase>360) phase -= 360;

                    //if(phase<0) phase += 180;

                    index = phase/10;

                    bucket[index] += mag;

                    //qDebug() << scale << " " << i << " " << x << " " << y << " " << x_t << " " << y_t << " " << index << mag;
                    //qDebug() << index;
                }
            }

            for(int b = 0; b<36; b++)
            {
                if(bucket[b]>max)
                {
                    max = bucket[b];
                    dir = b*10;
                }

                qDebug()<< b << " " << bucket[b];
            }

            //qDebug()<< dir << " " << max;

            for(int b = 0; b<36; b++)
            {
                if(bucket[b]>max*0.8&& b != dir)
                {
                    KeyPoint[scale].push_back({KeyPoint[scale][i].x,KeyPoint[scale][i].y,(double)b*10,bucket[b]});
                   //qDebug()<< b << " " << bucket[b];
                }

                //qDebug()<< b << " " << bucket[b];
            }


            KeyPoint[scale][i].direction = dir;
            KeyPoint[scale][i].magnitude = max;
            //qDebug()<< cnt << " " << dir << bucket[8] << bucket[9];
            //cnt++;


        }

    }


}
```


```C++
void KeyPoint_descriptor(std::vector<std::vector<real_key>>& KeyPoint, std::vector<KImageDouble> Octave_0,double sigma)
{

    KImageDouble Scale_img;
    int x,y;
    double mag[81];
    double phase[81];
    double gau_kenel;

    double bucket[8] = {0,};

    int x_t,y_t;




    //int cnt = 1;

    int KeyPoint_scale_size = 0;

    for(int scale = 0; scale<KeyPoint.size(); scale++) //root 2 sigma, 2 sigma
    {
        Scale_img = Octave_0[scale+1]; //origin roo2, origin 2
        sigma = sigma*pow(1.4,scale+1);
        KeyPoint_scale_size = KeyPoint[scale].size();
        for(int i=0;i<KeyPoint_scale_size;i++)
        {


            x = KeyPoint[scale][i].x;
            y = KeyPoint[scale][i].y;

            //11개 좌표 x+w_x, y+w_y
            for(int w_x = -4; w_x < 5; w_x++)
            {
                for(int w_y = -4; w_y < 5; w_y++)
                {
                    x_t = x+w_x;
                    y_t = y+w_y;

                    if(x_t>Scale_img.Row()-2||x_t<1||y_t>Scale_img.Col()-2||y_t<1) continue;
                    gau_kenel = 1./(2*M_PI*sigma*sigma) * exp(-0.5*(w_x*w_x + w_y*w_y)/(sigma*sigma));

                    mag[(w_x+4)*8+w_y+4] = gau_kenel * sqrt(pow(Scale_img[x_t+1][y_t]-Scale_img[x_t-1][y_t],2)
                                                            +pow(Scale_img[x_t][y_t+1]-Scale_img[x_t][y_t-1],2));

                    phase[(w_x+4)*8+w_y+4] = atan2(Scale_img[x_t][y_t+1]-Scale_img[x_t][y_t-1],
                                                    Scale_img[x_t+1][y_t]-Scale_img[x_t-1][y_t])*180/M_PI+180.0;

                }
            }

            //index = phase/45
            for(int p = 0; p < 2; p++)
            {
                std::vector<double> dou_vec;

                bucket[(int)phase[0+5*p]/45] += mag[0+5*p];
                bucket[(int)phase[1+5*p]/45] += mag[1+5*p];
                bucket[(int)phase[2+5*p]/45] += mag[2+5*p];
                bucket[(int)phase[3+5*p]/45] += mag[3+5*p];

                bucket[(int)phase[9+5*p]/45] += mag[9+5*p];
                bucket[(int)phase[10+5*p]/45] += mag[10+5*p];
                bucket[(int)phase[11+5*p]/45] += mag[11+5*p];
                bucket[(int)phase[12+5*p]/45] += mag[12+5*p];

                bucket[(int)phase[18+5*p]/45] += mag[18+5*p];
                bucket[(int)phase[19+5*p]/45] += mag[19+5*p];
                bucket[(int)phase[20+5*p]/45] += mag[20+5*p];
                bucket[(int)phase[21+5*p]/45] += mag[21+5*p];

                bucket[(int)phase[27+5*p]/45] += mag[27+5*p];
                bucket[(int)phase[28+5*p]/45] += mag[28+5*p];
                bucket[(int)phase[29+5*p]/45] += mag[29+5*p];
                bucket[(int)phase[30+5*p]/45] += mag[30+5*p];


                for(int b = 0;b<8;b++)
                {
                    dou_vec.push_back(bucket[b]);
                }

                KeyPoint[scale][i].feature.push_back(dou_vec);
            }


            for(int p = 0; p < 2; p++)
            {
                std::vector<double> dou_vec;

                bucket[(int)phase[45+5*p]/45] += mag[45+5*p];
                bucket[(int)phase[46+5*p]/45] += mag[46+5*p];
                bucket[(int)phase[47+5*p]/45] += mag[47+5*p];
                bucket[(int)phase[48+5*p]/45] += mag[48+5*p];

                bucket[(int)phase[54+5*p]/45] += mag[54+5*p];
                bucket[(int)phase[55+5*p]/45] += mag[55+5*p];
                bucket[(int)phase[56+5*p]/45] += mag[56+5*p];
                bucket[(int)phase[57+5*p]/45] += mag[57+5*p];

                bucket[(int)phase[63+5*p]/45] += mag[63+5*p];
                bucket[(int)phase[64+5*p]/45] += mag[64+5*p];
                bucket[(int)phase[65+5*p]/45] += mag[65+5*p];
                bucket[(int)phase[66+5*p]/45] += mag[66+5*p];

                bucket[(int)phase[72+5*p]/45] += mag[72+5*p];
                bucket[(int)phase[73+5*p]/45] += mag[73+5*p];
                bucket[(int)phase[74+5*p]/45] += mag[74+5*p];
                bucket[(int)phase[75+5*p]/45] += mag[75+5*p];


                for(int b = 0;b<8;b++)
                {
                    dou_vec.push_back(bucket[b]);
                }

                KeyPoint[scale][i].feature.push_back(dou_vec);
            }


        }

    }


}
```



```C++
void Mark_KeyPoint(KImageGray &igScale, double mag, double ori_deg, int pX, int pY){
    double dX_circle , dY_circle;
    int iX_circle, iY_circle;

    double radius = 0;

    int iX_dir, iY_dir;
    double dX_dir, dY_dir;

    double custom = 4;

    int tmp = (int)(mag / custom);
    if(tmp < 2 ){
        radius = 5;
    }
    else if(tmp < 5){
        radius = 7;
    }
    else if(tmp < 10){
        radius = 8;
    }
    else{
        radius = 10;
    }

    double theta_rad;
    //Draw circle
    for(int angle = 0; angle <= 360; angle+= 1){

        theta_rad = (double)angle * 0.01745329; // 0.017453..=>1 / 180 * 3.14592; degree to rad

        dX_circle = (double)pX - (double)(radius * cos(theta_rad));
        dY_circle = (double)pY - (double)(radius * sin(theta_rad));

        iX_circle = (int)dX_circle; iY_circle = (int)dY_circle;

        if(iX_circle > 0 && iY_circle > 0 && iX_circle < (int)igScale.Row() && iY_circle < (int)igScale.Col()){
           igScale[iX_circle][iY_circle] = 255;
        }
    }

    //Draw Direction
    for(int range = 0; range <= radius; range++){
        theta_rad = ori_deg * 0.01745329; // 0.017453..=>1 / 180 * 3.14592; degree to rad

        dX_dir = (double)pX - (double)(range * cos(theta_rad));
        dY_dir = (double)pY - (double)(range * sin(theta_rad));

        iX_dir = (int)dX_dir; iY_dir = (int)dY_dir;

        if(iX_dir > 0 && iY_dir > 0 && iX_dir < (int)igScale.Row() && iY_dir < (int)igScale.Col()){
           igScale[iX_dir][iY_dir] = 255;
        }

    }

}
```

```C++
typedef struct real{
    int x,y;
    double scale;
    double direction = -1;
    double magnitude = -1;
    std::vector<std::vector<double>> feature;
}real_key;
```
