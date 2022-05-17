# Ingress

## What is Ingress?
Mô hình hệ thống thuẩn túy:

![](..%5Cimages%5Cingress-modelclassic.png)

Trong mô hình này, chúng ta phải kiểm soát rất nhiều thành phần.
- Service
- LoadBlancer
- SSL 

Chúng ta không thể tạo từng Object trên với một hệ thống lớn.Vậy làm thế nào để kiểm soát tất cả các thành phần này trong Kubernetes Cluster? -> Ingress ra đời

Ingress sẽ đưa tất cả các thành phần nói trên vào trong một object của Kubernetes và có thể dễ dàng cấu hình như các object khác.
![](..%5Cimages%5Cingress-model.png)

Ingress giúp cho user truy cập ứng dụng bên trong Kubernetes Cluster thông qua URL duy nhất.
Và ta có thể tùy chỉnh cấu hình các đường dẫn URL khác nhau đến từng ứng dụng bên trong Kubernetes Cluster. 

Nhưng khi sử dụng Ingress thì ta vẫn phải expose ra ngoài Kubernetes Cluster thông qua Ingress-service (sử dụng NodePort hoặc Loadbalancer)

## How does Ingress control?

- Thực hiện loadbalancer như thế nào?
- Thực hiện gắn SSL như thế nào?

**Thành phần trong Ingress**
- Ingress Controller: Là các Pod sử dụng Image NGINX hoặc Hapory và cần route traffic từ pod này đến các service bên dưới trong cụm Kubernetes cluster.
- Ingress Resources: Để thiết lập các rule sẽ là các template định nghĩa phương thức làm việc cho Ingress (cấu hình route URL, cấu hình SSL certificate,...). (có định dạng giống Pod, Deployment,..)

![](..%5Cimages%5Cingress-tp.png)

Ingress không tổn tại mặc định trong Kubernetes Cluster.

**Phương thức deploy**

*Ingress Controller:*

Phương thức deploy Ingress controller dựa trên các nền tảng như:
- GCP HTTP(S) LoadBalancer (GCE) ~ *(Được K8S hỗ trợ nâng cấp)*
- NGINX ~ *(Được K8S hỗ trợ nâng cấp)*
- Contour
- HAProxy
- Traefik
- Istio

Ingress Nginx có thể làm các việc như:
- LoadBalancer Nginx
- Monitor Kubernetes Cluster
- Ingress resources
- Configure Nginx server.
- ...

**Ingress controller sử dụng Nginx là một deployment** trong Kubernetes. Sử dụng template dưới đây (Phương thức triển khai tôi sẽ nói ở bài sau)  [tham khảo](https://github.com/kubernetes/ingress-nginx/tree/main/deploy/static/provider/kind):
```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
      args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```
Để cấu hình được nginx thì cần khỏi tạo Configmap     
```yml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
```
Để deployment expose được ra ngoài cần tạo service:
```yml
apiVersion: v1
kind: Service
metadata: 
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```
Khởi tạo Service account với các thành phần dưới để quản lý Ingress controller
- Roles
- ClusterRoles
- RoleBindings
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```

*Ingress Resources*

Là thiết lập rule trên Ingress controller.

- Sample01:

![](..%5Cimages%5Cingress-model01.png)


- Sample02:

![](..%5Cimages%5Cingress-model02.png)

- Sample03:

![](..%5Cimages%5Cingress-model03.png)