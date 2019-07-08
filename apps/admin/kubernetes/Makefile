# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
BASEDIR = $(shell pwd)

include ../../../Makefile.properties


reset: clean.deployment app
	say "App Reset"

reset.safe:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o "$(BASEDIR)/../containers/main" "$(BASEDIR)/../containers/main.go" "$(BASEDIR)/../containers/kubernetes.go"
	gcloud builds submit "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/admin
	-kubectl delete deployment admin-deployment
	-kubectl run admin-deployment --image=$(DOCKERREPO)/admin --replicas=1 --port=8080 --labels=app=admin --env="APIIMAGE=$(DOCKERREPO)/api"
	say "app refresh complete"

app: build deploy

main:
	go get github.com/gorilla/mux
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o "$(BASEDIR)/../containers/main" "$(BASEDIR)/../containers/main.go" "$(BASEDIR)/../containers/kubernetes.go"

build: env main
	gcloud builds submit "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/admin

build.dockerhub: main
	docker build "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO):admin
	docker push $(DOCKERREPO):admin

build.generic: main
	docker build "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/whackapod-admin
	docker push $(DOCKERREPO)/whackapod-admin

deploy: env creds deployment service

deploy.minikube: rbac deployment service

deploy.minikube.dockerhub: rbac
	kubectl run admin-deployment --image=$(DOCKERREPO):admin --serviceaccount='wap-admin' --replicas=1 --port=8080 --labels=app=admin --env="APIIMAGE=$(DOCKERREPO):api"
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"

deploy.generic: rbac
	kubectl run admin-deployment --serviceaccount='wap-admin' --image=$(DOCKERREPO)/whackapod-admin --replicas=1 --port=8080 --labels=app=admin --env="APIIMAGE=$(DOCKERREPO)/whackapod-api"
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"

test:
	cd 	"$(BASEDIR)/../containers" && go test

rbac:
	kubectl create serviceaccount wap-admin
	kubectl create clusterrolebinding wap-admin --clusterrole=cluster-admin --serviceaccount=default:wap-admin

deployment:
	kubectl run admin-deployment  --serviceaccount='wap-admin' --image=$(DOCKERREPO)/admin --replicas=1 --port=8080 --labels=app=admin --env="APIIMAGE=$(DOCKERREPO)/api"

service:
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"

clean: env creds clean.deployment clean.service clean.rbac

clean.minikube: clean.deployment clean.service clean.rbac

clean.minikube.dockerhub: clean.deployment clean.service clean.rbac

clean.rbac:
	-kubectl delete serviceaccount wap-admin
	-kubectl delete clusterrolebinding wap-admin

clean.deployment:
	-kubectl delete deployment admin-deployment

clean.service:
	-kubectl delete service admin

clean.generic: clean.deployment clean.service clean.rbac

retry: clean build deploy

config: env
	echo "No custom config needed for admin"
