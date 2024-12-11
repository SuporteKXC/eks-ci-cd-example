
# Fluxo de CI/CD com GitHub, CodePipeline, CodeBuild e EKS

Essa documentação visa mostrar como funcionaria o workflow de deploys no ambiente EKS, utilizando como fonte qualquer provedor de controle de versão (GitHub, BitBucket, GitLab...).

### No repositório da aplicação
Vamos precisar ter os seguintes itens:
* [Dockerfile](Dockerfile);
* Arquivo de deployment com service, ingress, hpa e o que mais for relevante (No exemplo, [k8s-deployment.yaml](k8s-deployment.yaml);
* Arquivo de especificação de build do CodeBuild (No exemplo, [buildspec.yaml](buildspec.yaml));

### Na infraestrutura da AWS
Vamos ter uma pipeline no CodePipeline configurada para fazer o build da aplicação toda vez que for feita uma nova modificação na branch configurada.<br />

No arquivo buildspec.yaml, temos:<br />
```bash
docker build -t $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/$project_path:$VERSION --build-arg ACCOUNT_ID=$ACCOUNT_ID .
```
Que faz o build da Dockerfile, colocando a tag de acordo com o hash do commit feito.
<br />
<br />

Após feito o build e push para o ECR:
```bash
docker push $ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/$project_path:$VERSION
```
<br />

Fazemos o deploy para o devido Cluster/Namespace/Deployment.<br />
Neste primeiro comando, fazemos o AssumeRole para uma Role que possui acesso ao Cluster Kubernetes:
```bash
aws eks update-kubeconfig --region us-east-1 --name $CLUSTER --role-arn arn:aws:iam::$ACCOUNT_ID:role/EksCodeBuildkubectlRole
```
<br />
<br />

```bash
cat k8s-deployment.yml | envsubst | kubectl apply -f -
```

Enquanto este outro, aplica o arquivo de Deployment, fazendo a <b>subsituição de váriaveis, que estão declaradas direto no projeto do CodeBuild<b/>.