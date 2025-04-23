# Technical-Assessment-Submission

# DevOps Assessment Task

# Step 1: Docker - Ruby on Rails with PostgreSQL

# Rails App Creation:  rails new myapp -d postgresql
cd myapp

# Dockerfile (in Rails app root):

FROM ruby:3.1

RUN apt-get update -qq && \
    apt-get install -y nodejs postgresql-client

WORKDIR /myapp
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install

COPY . /myapp

EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]

# docker-compose.yml:
version: '3'
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: password
    volumes:
      - pgdata:/var/lib/postgresql/data

  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && rails db:create && rails db:migrate && rails s -b '0.0.0.0'"
    volumes:
      - .:/myapp
    ports:
      - "3000:3000"
    depends_on:
      - db

volumes:
  pgdata:

# Kubernetes
# PostgreSQL StatefulSet:

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        env:
        - name: POSTGRES_PASSWORD
          value: password
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: pgdata
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi

Rails Deployment and Service:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rails-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rails-app
  template:
    metadata:
      labels:
        app: rails-app
    spec:
      containers:
      - name: rails-app
        image: <your-dockerhub-username>/rails-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_HOST
          value: postgres
        - name: DATABASE_PASSWORD
          value: password
---
apiVersion: v1
kind: Service
metadata:
  name: rails-service
spec:
  type: ClusterIP
  selector:
    app: rails-app
  ports:
    - port: 3000
      targetPort: 3000

Ingress:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rails-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: rails.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rails-service
            port:
              number: 3000


Step 3: ArgoCD
1.	application.yaml:
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: rails-app
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  source:
    repoURL: https://github.com/<your-username>/<repo-name>
    targetRevision: HEAD
    path: manifests
  project: default
  syncPolicy:
    automated:
      selfHeal: true
                prune: true



# argocd-cm.yaml and argocd-rbac-cm.yaml:
# repo-secret.yaml:

apiVersion: v1
kind: Secret
metadata:
  name: repo-creds
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  url: https://github.com/<your-username>/<repo-name>
  password: <your-password>
  username: <your-username>
  type: git


# Tekton CI/CD
# Pipeline.yaml (basic example):
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-and-push
spec:
  steps:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:latest
      args:
        ["--dockerfile=/workspace/source/Dockerfile", "--context=/workspace/source/", "--destination=docker.io/<your-user>/rails-app:latest"]
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: rails-ci-pipeline
spec:
  tasks:
    - name: build-and-push
      taskRef:
name: build-and-push



# Conclusion
This assessment task provided a comprehensive walkthrough of implementing a full-stack DevOps pipeline using modern tools and technologies. From containerizing a Ruby on Rails application with Docker to orchestrating services with Kubernetes, automating deployments with ArgoCD, and streamlining CI/CD processes using Tekton, the project showcases a complete deployment lifecycle. Successfully completing this exercise demonstrates practical knowledge in DevOps best practices, infrastructure as code, containerization, and continuous integration and delivery pipelines. It lays a solid foundation for managing scalable, automated, and reliable application deployments in real-world environments.
# GitHub Repo:  https://github.com/vibin3021/Technical-Assessment-Submission/tree/main
