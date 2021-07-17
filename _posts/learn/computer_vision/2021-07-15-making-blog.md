---
layout: post
title:  "[Computer_Vision] 5. Canny Edge Operator"
subtitle:   "정문호 교수님"
categories: 
    - learn
    - computer_vision
tags: [computer_vision]
comments: true
use_math: true
---

# 5. Canny Edge Operator

> # Canny Edge Operator
> ## Input Image

![InputImg](https://user-images.githubusercontent.com/69707792/125769569-c4962fa9-d039-4ec9-b931-a085f3a085c0.JPG)


> ## Canny Edge Operator

1. 노이즈 제거를 위한 smoothing
2. Sobel mask를 이용한 edge 후보 찾기
3. Non-Maxima Suppression 
4. Hysteresis Thresholding



+ 2. Sobel mask를 이용한 edge 후보 찾기
    - magnitude를 이용해 edge의 direction을 구한다.
    
+ 3. Non-Maxima Suppression   
    - Local Maxima를 선택해서 얇은 edge를 구할 수 있게 하는 과정
    - 2에서 구한 direction을 통해서 edge 후보의 양옆과 비교를 해서 최대인 경우 남김
+ 4. Hysteresis Thresholding
    - edge의 끊김 방지를 위해 Low Threshold를 넘고 주변에 edge가 있다면 edge로 

> ### result

![resultImg](https://user-images.githubusercontent.com/69707792/125769601-b09b022e-3d3c-4c44-a1b1-aeec9b4fe6f3.JPG)


> ### Code
```C++
void MainFrame::on_button_CannyEdge_clicked()
{

    KImageGray icMain;
    KImageGray icMain2;

    //포커스 된 ImageForm으로부터 영상을 가져옴
    if(_q_pFormFocused != 0 && _q_pFormFocused->ImageGray().Address() &&  _q_pFormFocused->ID() == "OPEN")
    {
        icMain = _q_pFormFocused->ImageGray();
        icMain2 = _q_pFormFocused->ImageGray();
    }
    else
        return;

    int row = icMain.Row();
    int col = icMain.Col();

    //Gaussian smoothing
    double sigma = ui->SpinBox_sigma->value();

    int Gau_filter_size = 8*sigma + 1; //filter size

    int size = std::sqrt(Gau_filter_size);

    //qDebug()<<size;

    double mask;


    for(int ii = size; ii<row-size;ii++){
        for(int jj = size; jj<col-size;jj++){

            mask = 0;


            for(int i = -size; i<size+1; i++){
                for(int j = -size; j<size+1;j++){
                    mask += std::exp(-0.5*((i*i+j*j)/(sigma*sigma)))*icMain[ii-i][jj-j];

                }
            }

            icMain[ii][jj] = (1/(2*M_PI*sigma*sigma))*mask;
        }
    }

    //Sobel Mask

    int dLow = 80;
    int dHigh = 120;
    double mag_x;
    double mag_y;
    double phase;

    double** magnitude = new double*[row]{0,};
    int** direction = new int*[row] {0,};
    for(int i = 0; i < row; i++)
    {
        magnitude[i] = new double[col]{0,};  // magnitude
        direction[i] = new int[col]{0,};
    }

    double Mask_w[3][3] = { {-1., 0., 1.}, {-2., 0., 2.}, {-1., 0. ,1.} };
    double Mask_h[3][3] = { {1., 2., 1.}, {0., 0., 0.}, {-1., -2. ,-1.} };

    //qDebug()<<row<< " "<<col;


    qDebug()<<(unsigned char)((((int)(113/22.5)+1)>>1) & 0x00000003);

    for(int ii = 1; ii<row-1;ii++){
        for(int jj = 1; jj<col-1;jj++){

            mag_x = 0;
            mag_y = 0;

            for(int i = -1; i<2; i++){
                for(int j = -1; j<2;j++){
                    mag_x += (Mask_w[i+1][j+1]*icMain[ii+i][jj+j]);
                    mag_y += (Mask_h[i+1][j+1]*icMain[ii+i][jj+j]);
                }
            }

            magnitude[ii][jj] = fabs(mag_x) + fabs(mag_y);

            if(magnitude[ii][jj] > dLow)
            {
                //icMain2[ii][jj] = 255;
                phase = atan2(mag_x,mag_y)*180/M_PI;
                if(phase>360) phase -= 360;

                if(phase<0) phase += 180;

                direction[ii][jj] = (unsigned char)((((int)(phase/22.5)+1)>>1) & 0x00000003);


                qDebug()<<ii<<" "<<jj<<" "<<phase<<" "<<direction[ii][jj];


            }
            else {
                magnitude[ii][jj] = 0;
                //icMain2[ii][jj] = 0;
            }

        }
    }


    // Non-Maxima Suppression
    //int nDx[4] = {0, 1, 1, 1};
    //int nDy[4] = {1, 1, 0, -1};
    int nDx[4] = {0, 1, 1, -1};
    int nDy[4] = {1, 1, 0, 1};

    double** buffer = new double*[row] {0,};
    int** realedge = new int*[row]{0,};
    for(int i = 0; i < row; i++)
    {
        buffer[i] = new double[col]{0,};
        realedge[i] = new int[col]{0,};
    }

    class HighEdge{
        public:
            int x;
            int y;
    };

    HighEdge h;
    std::stack<HighEdge> highedge;

    for(int ii = 1; ii<row-1;ii++){
        for(int jj = 1; jj<col-1;jj++){
            if(magnitude[ii][jj] == 0) continue;

            //if(magnitude[ii][jj] > magnitude[ii+nDx[direction[ii][jj]]][jj+nDy[direction[ii][jj]]] && magnitude[ii][jj] > magnitude[ii-nDx[direction[ii][jj]]][jj-nDy[direction[ii][jj]]])
            if(magnitude[ii][jj] > magnitude[ii+nDy[direction[ii][jj]]][jj+nDx[direction[ii][jj]]] && magnitude[ii][jj] > magnitude[ii-nDy[direction[ii][jj]]][jj-nDx[direction[ii][jj]]])
            {
                if(magnitude[ii][jj] > dHigh)
                {
                    h.x = ii;
                    h.y = jj;

                    highedge.push(h);
                    realedge[ii][jj] = 255;

                }

                //realedge[ii][jj] = 255;
                buffer[ii][jj] = magnitude[ii][jj];
            }
        }
    }


    for(int ii = 1; ii<row-1;ii++){
        for(int jj = 1; jj<col-1;jj++){
            //icMain2[ii][jj] = realedge[ii][jj];
        }
    }


    //Thresholding
    int x,y;
    while(!highedge.empty())
    {
        x = highedge.top().x;
        y = highedge.top().y;
        for(int i = -1; i < 2; i++)
        {
            for(int j = -1; j < 2; j++)
            {
                if(buffer[x+i][y+j] && buffer[x+i][y+j]<=dHigh)
                {
                    h.x = x+i;
                    h.y = y+j;
                    highedge.push(h);
                    realedge[x+i][y+j] = 255;
                    buffer[x+i][y+j] = 0;
                }
            }
        }
        highedge.pop();
    }

    for(int ii = 1; ii<row-1;ii++){
        for(int jj = 1; jj<col-1;jj++){
            icMain2[ii][jj] = realedge[ii][jj];
        }
    }

    //Show
    ImageForm*  q_pForm1 = new ImageForm(icMain2, "Canny Edge Operator", this);
    _plpImageForm->Add(q_pForm1);
    q_pForm1->show();

}
```

