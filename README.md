# Instruções

- **Requisistos:** 
    - kubectl
    - Docker
    - kind

### 1. Criar um cluster local
- Crie um arquivo de configuração do cluster
```bash
touch openfaas-cluster.yml
```
- Insira o conteúdo:
```yml
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

- Use o comando para criar o cluster:
```bash
kind create cluster --config openfaas-cluster.yml
```
### 2. Instalar OpenFaas

- Instalar arkade
```bash
curl -sSL https://get.arkade.dev | sudo -E sh
```

- Instalar OpenFaas
```bash
arkade install openfaas
```

- Intalar faas-cli
```bash
arkade get faas-cli
```

- Execute:
```bash
kubectl rollout status -n openfaas deploy/gateway
kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```

### 3. Criar uma função
- Crie uma pasta para a função
```bash
mkdir functions && cd functions
```

- Criar função utilizando faas-cli
```bash
faas-cli new --lang go hello-go
```
- Arquivos criados:
    - hello-go/go.mod
    - hello-go/handler.go
    - hello-go.yml
    - template/

- **Obs:** O diretório template é criado para ser utilizado no deploy da função

- Observe o arquivo hello-go/handler.go
```go
// Handle a serverless request
func Handle(req []byte) string {
	return fmt.Sprintf("Hello, Go. You said: %s", string(req))
}
```
Essa é a função que será executada

- Observe o arquivo hello-go.yml
```yml
version: 1.0
provider:
  name: openfaas
  gateway: http://127.0.0.1:8080
functions:
  hello-go:
    lang: go
    handler: ./hello-go
    image: hello-go:latest
```
- Modifique a tag image para:
```yml
image: <dockerhub-user>/hello-go:latest
```

- Construindo a função
```bash
faas-cli build -f ./hello-go.yml
```

- Com a função construída, ela será exibida nas imagens do docker
```bash
docker images | grep hello-go
```

- Faça login no dockerhub
```bash
docker login --username <username>
```

- Faça o push da imagem para o dockerhub
```bash
sudo faas-cli push -f ./hello-go.yml
```

- Faça no login no gateway:
-
```bash
 export PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
```
```bash
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

- Deploy da função:
```bash
faas-cli deploy -f ./hello-go.yml
```

- Para executar a função:
```bash
curl localhost:8080/function/hello-go -d "Testando a função"
```
