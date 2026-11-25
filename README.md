# SEG3502 Lab 10 - configuration files

Copy these files into your project, keeping this layout:

    containersLab/
      books-rest-api/        <- Dockerfile + src/main/resources/application.properties
      bookstore-client/      <- Dockerfile, entrypoint.sh, src/main.ts, src/assets/env*.json
      docker/docker-compose.yml
      k8s/*.yaml

## One manual edit (not included)
In bookstore-client/src/app/.../books.service.ts, replace the Url field with:

    private Url = ((window as { [key: string]: any })['env']).apiUrl;

Make sure src/assets is listed in the "assets" of angular.json so env.template.json is copied.

## Order of commands
Docker (Q1-Q2):
    docker image build --network=host -t books-rest-api:latest books-rest-api
    docker network create mysql-network
    docker container run -d --name mysqldb --network mysql-network -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=booksDb mysql:9
    docker container run -d --network mysql-network --name booksserver -e MYSQL_USER=root -e MYSQL_PASSWORD=root -e MYSQL_DB=booksDb -e MYSQL_HOST=mysqldb -p 8080:8080 books-rest-api:latest
    docker container ls                      # Q1
    docker image build -t books-api-client:latest bookstore-client
    docker container run -d --name booksclient -e API_URL='http://localhost:8080/books-api/' -p 4100:80 books-api-client:latest
    docker container ls                      # Q2

Compose (Q3): stop/remove the containers above first, then in docker/:
    docker compose up -d --build             # Q3
    docker compose down

Kubernetes (Q4-Q6), in k8s/:
    minikube start
    minikube image load books-rest-api:latest
    minikube image load books-api-client:latest
    kubectl apply -f .                       # Q4
    kubectl get deployments                  # Q5
    kubectl port-forward service/booksserver 8080:8080   (own terminal)
    kubectl port-forward service/booksapp 4100:4100      (own terminal)
    kubectl get pods                         # Q6
    kubectl delete pod <booksserver-pod>; kubectl get pods
    kubectl delete -f .

## Deviations from the handout
- Dockerfile uses ./gradlew (the handout's "gradlew" is not on PATH in the Alpine image).
- k8s MySQL image is mysql:9 (handout YAML said 8, text said 9; compose uses 9).
- booksserver has replicas: 2 (handout text and Q6 assume two pods; YAML said 1).
