# Deploy machine learning models in production

Cortex is an open source platform for deploying machine learning models—trained with nearly any framework—as production web services.

<br>

<!-- Set header Cache-Control=no-cache on the S3 object metadata (see https://help.github.com/en/articles/about-anonymized-image-urls) -->
![Demo](https://d1zqebknpdh033.cloudfront.net/demo/gif/v0.8.gif)

<br>

## Key features

* **Autoscaling:** Cortex automatically scales APIs to handle production workloads.
* **Multi framework:** Cortex supports TensorFlow, PyTorch, scikit-learn, XGBoost, and more.
* **CPU / GPU support:** Cortex can run inference on CPU or GPU infrastructure.
* **Spot instances:** Cortex supports EC2 spot instances.
* **Rolling updates:** Cortex updates deployed APIs without any downtime.
* **Log streaming:** Cortex streams logs from deployed models to your CLI.
* **Prediction monitoring:** Cortex monitors network metrics and tracks predictions.
* **Minimal configuration:** Deployments are defined in a single `cortex.yaml` file.

<br>

## Spinning up a Cortex cluster

Cortex is designed to be self-hosted on any AWS account. You can spin up a Cortex cluster with a single command:

```bash
$ cortex cluster up

aws region: us-west-2
aws instance type: p2.xlarge
min instances: 0
max instances: 10
spot instances: yes

￮ spinning up your cluster ...
your cluster is ready!
```

<br>

## Deploying a model

### Implement your predictor

```python
# predictor.py

model = download_model()

def predict(sample, metadata):
    return model.predict(sample["text"])
```

### Configure your deployment

```yaml
# cortex.yaml

- kind: deployment
  name: sentiment

- kind: api
  name: classifier
  predictor:
    path: predictor.py
  tracker:
    model_type: classification
  compute:
    gpu: 1
    mem: 4G
```

### Deploy to AWS

```bash
$ cortex deploy

creating classifier (http://***.amazonaws.com/sentiment/classifier)
```

### Serve real-time predictions

```bash
$ curl http://***.amazonaws.com/sentiment/classifier \
    -X POST -H "Content-Type: application/json" \
    -d '{"text": "the movie was amazing!"}'

positive
```

### Monitor your deployment

```bash
$ cortex get classifier --watch

status   up-to-date   available   requested   last update   avg latency
live     1            1           1           8s            24ms

class     count
positive  8
negative  4
```

<br>

## What is Cortex an alternative to?

Cortex is an open source alternative to serving models with SageMaker or building your own model deployment platform on top of AWS services like Elastic Kubernetes Service (EKS), Elastic Container Service (ECS), Lambda, Fargate, and Elastic Compute Cloud (EC2) or open source projects like Docker, Kubernetes, and TensorFlow Serving.

<br>

## How does Cortex work?

The CLI sends configuration and code to the cluster every time you run `cortex deploy`. Each model is loaded into a Docker container, along with any Python packages and request handling code. The model is exposed as a web service using Elastic Load Balancing (ELB), TensorFlow Serving, and ONNX Runtime. The containers are orchestrated on Elastic Kubernetes Service (EKS) while logs and metrics are streamed to CloudWatch.

<br>

## Examples of Cortex deployments

<!-- CORTEX_VERSION_README_MINOR x5 -->
* [Sentiment analysis](https://github.com/cortexlabs/cortex/tree/0.11/examples/tensorflow/sentiment-analyzer): deploy a BERT model for sentiment analysis.
* [Image classification](https://github.com/cortexlabs/cortex/tree/0.11/examples/tensorflow/image-classifier): deploy an Inception model to classify images.
* [Search completion](https://github.com/cortexlabs/cortex/tree/0.11/examples/tensorflow/search-completer): deploy Facebook's RoBERTa model to complete search terms.
* [Text generation](https://github.com/cortexlabs/cortex/tree/0.11/examples/pytorch/text-generator): deploy Hugging Face's DistilGPT2 model to generate text.
* [Iris classification](https://github.com/cortexlabs/cortex/tree/0.11/examples/sklearn/iris-classifier): deploy a scikit-learn model to classify iris flowers.
