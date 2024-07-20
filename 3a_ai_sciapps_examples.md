# PEARC24 Kubernetes Tutorial

AI Examples\
Hands on session

## AI training using PyTorch example

You can use the yaml files from the repo as is but please change the username to something unique as we are all sharing the same namespace. We start by running the training example:

```
kubectl apply -f pytorch-training.yaml
```
Check on status:

```
kubectl get pods
```

When the job is in run state you can check the logs for output:

```
kubectl logs gp3-username
```

Once you are done exploring, please delete the pod:

```
kubectl delete -f pytorch-training.yaml
```

## Text generation inference example

Start up the inference pod:

```
kubectl apply -f tgi-inference.yaml
```

Once the pod is running, get interactive access to the pod:

```
kubectl exec -it tgi-username -- /bin/bash
```

Once in the pod, start a python3 interpreter and then run:

```
from huggingface_hub import InferenceClient
client = InferenceClient(model="http://0.0.0.0:80")
or token in client.text_generation("Who made cat videos?", max_new_tokens=24, stream=True): print (token)
```
## RAG example using Ollama

Start up the pod:
```
kubectl apply -f ollama-rag.yaml
```
Watch the logs and make sure you wait till the installs are done and the book is downloaded:

```
kubectl logs ollama-username
```
Once the book is downlaoded (you will see wget output in the logs), we can get interactive access to the pod and start up the Ollama server and pull the module we want to use (Mistral):

```
kubectl exec -it ollama-username -- /bin/bash
cd /scratch
nohup ollama serve&
ollama pull mistral
```
We can now download our test script and run it:
```
wget https://raw.githubusercontent.com/mahidhar/5nrp_k8s_tutorial/main/test.py
python3 -i test.py
```
Now we can run the rag within the interactive python interpreter. Do the following one by one (i.e. wait for results before moving to the next one)
```
rag.invoke("What do you feed pigeons ?")
rag.invoke("Do tame pigeons have better plumage ?")
rag.invoke("What affects pigeon plumage ?")
```

## Helm based deployment of LLM as service (H2O)

Install helm. Details at: <https://github.com/helm/helm#install>. Quickest option mignt be to download and use static binaries from the release page referenced in link above.

We will be using the H20 project (<https://github.com/h2oai>). Clone the H2O repository:

```
git clone https://github.com/h2oai/h2ogpt.git
```

Copy the values file to the h2o directory:

```
cp h2o-values.yaml h2ogpt
cd h2ogpt
```
Install the helm chart. Make sure you use a unique name (change username below):

```
helm install h2ogpt-username helm/h2ogpt-chart -f h2o-values.yaml
```

Check the pods (kubectl get pods, maybe grep your username since there will be a lot of pods running) to make sure they are running. Once the pod is running, check the logs and see if h2o is running. Check if the service is running and then port forward to your laptop:

```
kubectl check service
kubectl port-forward service/h2ogpt-username-web --address=127.0.0.1 16002:80
```

You can open localhost:16002 in your browser once the forward works. This will give you the H2O interface for chat, ingesting docs etc.

## LAMMPS (molecular dynamics code) example

Lets use an existing LAMMPS (a molecular dynamics code) container from dockerhub. 

###### lammps.yaml:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: lammps-<username>
spec:
  template:
    spec:
      volumes:
          - name: scratch
            emptyDir: {}
      containers:
      - name: test
        image: lammps/lammps:patch_7Jan2022_rockylinux8_openmpi_py3
        command: ["/bin/bash", "-c"]
        args:
        - >-
            cd /scratch;
            curl -O https://www.lammps.org/bench/inputs/in.lj.txt ;
            export OMP_NUM_THREADS=1;
            lmp_serial < in.lj.txt ;
            mpirun -np 4 lmp_mpi < in.lj.txt;
        volumeMounts:
            - name: scratch
              mountPath: /scratch
        resources:
          limits:
            memory: 16Gi
            cpu: "4"
            ephemeral-storage: 10Gi
          requests:
            memory: 16Gi
            cpu: "4"
            ephemeral-storage: 10Gi
      restartPolicy: Never
```
Lets run this simple application test:

```
kubectl apply -f test-lammps.yaml
```
Now lets check the output:
```
kubectl logs lammps-mahidhar-cj25r
```

## End

**Please make sure you did not leave any running pods. Jobs and associated completed pods are OK.**

