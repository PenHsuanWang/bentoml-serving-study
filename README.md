# Bentoml Model Serving for MLOps solution.

The following study note is releated with MLOps solution.
The Bentoml is an open-source project for MLOps. Check out the official webpage and github.

:point_right: https://docs.bentoml.org/en/latest/
:point_right: https://github.com

Here is starting from the point to serving model.
To do serving mechnism testing and deploy by Yatai(やたい) to K8S.
Before you go through this note, pre-reqiure knowledge about Bento and Model build can take following materials and reference.

:bulb: **preparing ML model:** https://docs.bentoml.org/en/latest/concepts/model.html

:bulb: **bentoml model serving:** https://docs.bentoml.org/en/latest/concepts/service.html

:bulb: **bemtoml runner:** https://docs.bentoml.org/en/latest/concepts/runner.html

:bulb: **over view picture of MLOps solution with BentoML:**
https://towardsdatascience.com/10-ways-bentoml-can-help-you-serve-and-scale-machine-learning-models-4060f1e59d0d

## Before Get Start

### Motivation
To work on a ML related project, several teams need to collaborate to build model from research to deployment in production environment.

#### research team:
Data Scientist and Analysist are put in charged for study data insight. Training and tuning the models.

#### deployment and operation:
Once the model performacne pass the testing criteria. Deployment in production and responsible for DevOps teams.

A model repository can help these teams cowork fluently. Here is my picture to use `MLflow` to build up the plateform. 

![](https://i.imgur.com/X6ev5PN.png)

![](https://i.imgur.com/yxlmTj7.png)

And the dockerfile can take my github repo as reference.  
:link: https://github.com/PenHsuanWang/mlflow-docker-setup


### My Picture for Integration Model Registry to BentoML

The workflow to integrage BentoML with MLflow.

![](https://i.imgur.com/qM7crjH.png)

BentoML provided model extraction api to retrive mlflow repository.

:bulb: [bentoml official docs](https://docs.bentoml.org/en/latest/reference/frameworks/mlflow.html)



## Run Model Serving with BentoML

## Adapting Batching

The Bentoml servering using multiprocess to execute API logic. The proven of programe process can be check from [ref]().

The architecture provided in the official document:
![](https://i.imgur.com/9vMF3Dn.png)

here we using jmeter to post full throttle performance test.

To test the number of api server shown in the atchitecture mentioned above. Using `bentoml` cli to run service.
```
bentoml serve service.py:svc --production --api-workers 8
```

In this case will run 8 processes to deal the API logic (generally doing feature angineering while inference request with raw data)

:::info
:bulb: **Hint:** The default value is the number of available CPU cores in production mode.
:::

## PyTorch mnist model
Here using official example to do the test. We usign **pytorch mnist**.

:point_right: **official example list** [bentoml github link](https://github.com/bentoml/BentoML/tree/main/examples).  

:point_right: **pytorch mnist example** [pytorch mnist project demo](https://github.com/bentoml/BentoML/tree/main/examples/pytorch_mnist)

### prerequest

* Installing all the requirement to train the pytorch model.
    ```pip install -r ./requirements.txt```
* Train the model and save to bentoml model store. Just run ```python train.py```
* Checking the model ```bentoml models list```

![](https://i.imgur.com/TtVxid7.png)

::: info
:spiral_note_pad: 
**Note:**  The BentoML model store can be found at local machine.  

![](https://i.imgur.com/oX5Rp1l.png)
:::

### Run the service

After check the model is exist in the bentoml model store.



run bentoml cli to execute the `service.py` to start.
```
bentoml serve service.py:svc --production --api-workers 8
```

![](https://i.imgur.com/gXOtqKH.png)


### Quick Test the bentoml service

Bentoml has been expose the api endpoint at `localhost:3000/<api_endpoint>`

:::info
:bulb: **Hint** : 
The _<api_endpoint>_ is the same as the function name defined at service.py
:::

To make the performance more distinctable between single process and multiprocess. I stop the thread for 1 second to mimic the time consumed during api logic processing.

> Demonstration of post REST API to post  Bentoml serving request.
![](https://i.imgur.com/U3HNiqd.gif)

### To Test the Architecture provided by official doc




## Full throttle test



