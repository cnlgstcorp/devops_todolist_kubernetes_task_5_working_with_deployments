Розгортання Django ToDo App у Kubernetes
Цей проєкт демонструє розгортання Django-застосунку зі списком справ у кластері Kubernetes із використанням Deployment, HPA, запитів/лімітів ресурсів і стратегії оновлення RollingUpdate.

📦 Підготовка
Створити неймспейс:

bash
kubectl create namespace mateapp
kubectl config set-context --current --namespace=mateapp
Застосувати Deployment:

bash
kubectl apply -f infrastructure/deployment.yml
Застосувати Horizontal Pod Autoscaler (HPA):

bash
kubectl apply -f infrastructure/hpa.yml

yaml
requests:
  cpu: "30m"
  memory: "64Mi"
limits:
  cpu: "250m"
  memory: "128Mi"
Значення вибрано на основі типової роботи Django у простої.

requests — мінімальний гарантований ресурс для стабільного запуску.

limits — верхня межа споживання, щоб уникнути перевантаження ноди.

Така конфігурація відповідає QoS класу Burstable, що дозволяє ефективно керувати ресурсами в кластері.

2. 🔁 Strategy: RollingUpdate
yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
Вибрана стратегія забезпечує безперервний доступ до застосунку.

maxUnavailable: 1: максимум 1 под може бути недоступним під час оновлення.

maxSurge: 1: дозволяє створити 1 додатковий под під час оновлення.

Підходить для сервісів, які мають бути доступні 24/7.

3. 📈 HPA (Horizontal Pod Autoscaler)
minReplicas: 2 — забезпечує доступність і відмовостійкість.

maxReplicas: 5 — дозволяє гнучко масштабувати під навантаження.

metrics: cpu + memory — автоскейлінг реагує на реальне споживання ресурсів.

HPA використовує формулу:
DesiredReplicas = CurrentReplicas × (CurrentMetricValue / TargetMetricValue)
Якщо використовується NodePort сервіс:

bash
kubectl port-forward svc/kube2py-service 8000:8080
Далі відкрий у браузері:

http://localhost:8000
Після деплою перевірити:

bash
kubectl get pods
kubectl get hpa
kubectl get deployment