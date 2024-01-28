# DuLa-Net
This is the pytorch demo code of our CVPR 2019 paper  
**DuLa-Net: A Dual-Projection Network for Estimating Room Layouts from a Single RGB Panorama ([Arxiv], [Project])**

By this repo you can estimate the 3D room layout from a single indoor RGB panorama. To see more details please refer to the paper or project page.

<img src='figs/system.jpg' width=100%>

## Prerequisites
- Python3
- Pytorch (CUDA >= 8.0)
- OpenCV-Python
- Pillow / scikit-image

## Pretrained Model

First, please download the [pretrained models] and copy to ./Model/ckpt/ \
The pretrained models are trained on our Realtor360 dataset with different backbone networks.

## Quick start

- Install packages

```
# https://pytorch.org/
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

pip install --no-cache-dir -r requirements.txt
```

- Start redis, celery and api server

```
docker run --rm -p 6379:6379 --name my-redis-container redis:latest

# For window
celery -A tasks worker --pool=solo -l info

#　Unix-like systems
celery -A tasks worker --loglevel=info

python .\server.py
```

## APIs

Explore detailed information about the APIs in the Swagger UI documentation, accessible at http://localhost:5000. The Swagger UI provides a comprehensive overview of the available APIs, including endpoints, methods, and detailed descriptions to assist you in understanding and interacting with the application.

In this repository, we've concentrated on building the AI service segment shown in the diagram below.

```mermaid
sequenceDiagram

    box User
    participant Frontend
    participant Data-Server
    end 
    
    box AI-Service
    participant AI-Server
    participant DuLa-Net
    participant Storage
    end

    Frontend->>+AI-Server: POST Task: { image, callbackURL(optional) }
    AI-Server ->>+ Storage: Create Folder: { id }
    Storage->>- AI-Server: Success
    AI-Server-->> Data-Server: Callback to update: { id, status:"initial" }
    AI-Server->> Frontend: Success: { id }
    AI-Server->>+DuLa-Net: Schedule Task: { id, image }
    DuLa-Net->>+Storage: Store Result: { id, image, result, metadata }
    Storage->>- DuLa-Net: Success
    DuLa-Net->>- AI-Server: Task Done: { result, status }
    AI-Server-->>- Data-Server: Callback to update: { id, result, status:"done" }

    Frontend->>+Data-Server: GET Task: { id }
    Data-Server->>-Frontend: { result, status }

    Frontend->>+AI-Server: Get Task: { id }
    AI-Server->>+Storage: Serve Static Files: { id }
    Storage->>-AI-Server: Success
    AI-Server->>-Frontend: Success: { id, links }
```



## Pre-processing

The input panorama should be already aligned with the Manhattan World. We recommand you using the [PanoBasic] in Matlab or the python implementation [here]. Those tool can help you do the pre-processing to align the panorama.

```
python preprocess.py --img_glob assets/raw/panorama.jpg --output_dir assets/preprocessed/
```

## Predict

Then using below command to load the pretrained model and predict the 3D layout.

```
python demo.py --input figs\001.jpg

# with docker
docker run --rm --gpus all dula-net demo.py --input figs\001.jpg
```
<img src='figs/001.jpg' width=50%><img src='figs/001_vis.jpg' width=50%>

If you want to use other backbone networks(default is resnet18).
```
python demo.py --input figs\001.jpg --backbone resnet50 --ckpt Model\ckpt\res50_realtor.pkl
```

## More Results

<img src='figs/002.jpg' width=50%><img src='figs/002_vis.jpg' width=50%>
<img src='figs/003.jpg' width=50%><img src='figs/003_vis.jpg' width=50%>
<img src='figs/004.jpg' width=50%><img src='figs/004_vis.jpg' width=50%>


## Dataset
The Realtor360 dataset currently couldn’t be made publicly available due to some legal privacy issue. Please refer to the MatterportLayout dataset(coming soon), which resembles the Realtor360 in all aspects.

[PanoBasic]: <https://github.com/yindaz/PanoBasic>
[here]: <https://github.com/sunset1995/HorizonNet/blob/master/preprocess.py>
[pretrained models]: <https://drive.google.com/file/d/1AJ-RsVW8XTvlaJ1hzGIMu4L2aDiEWc4z/view?usp=sharing>
[Arxiv]: <https://arxiv.org/abs/1811.11977>
[Project]: <https://cgv.cs.nthu.edu.tw/projects/dulanet>