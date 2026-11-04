# Práctica 5: persistir los logs de la calculadora

### Propósito

Solicitar almacenamiento persistente mediante un `PersistentVolumeClaim` (PVC),
montarlo en el backend de la calculadora y demostrar que los logs sobreviven al
reemplazo de un Pod. El Deployment usa una estrategia `Recreate` porque el PVC
`ReadWriteOnce` pertenece a un único nodo a la vez.

Esta práctica usa el addon CSI Hostpath de Minikube y la StorageClass
`csi-hostpath-sc`, útil con el clúster multinodo creado para el curso.

## Crear recursos

```bash
minikube addons enable csi-hostpath-driver
cd ../calculadora-backend
docker build -t calculadora-backend:latest .
minikube image load calculadora-backend:latest
cd ../practice-5
kubectl apply -f namespace.yaml
kubectl apply -f calculadora-pvc.yaml
kubectl apply -f calculadora-backend.yaml
kubectl apply -f calculadora-service.yaml
kubectl rollout status deployment/calculadora-backend -n practice-5
kubectl get pvc,pods -n practice-5
```

Para usar un registro externo, publica `tu-registro/calculadora-backend:latest`
y reemplaza esa imagen en `calculadora-backend.yaml`.

El PVC debe mostrar `Bound`. Genera una operación y deja un log persistente:

```bash
kubectl port-forward service/calculadora-backend 8080:8080 -n practice-5
```

En otra terminal:

```bash
curl "http://localhost:8080/api/v1/calculadora/multiplicar?numero1=6&numero2=8"
```

Lee el archivo desde el contenedor backend:

```bash
kubectl exec deployment/calculadora-backend -n practice-5 -- cat /app/logs/api-requests.txt
```

Elimina el Pod y espera su reemplazo; después verifica de nuevo el archivo:

```bash
kubectl delete pod -l app.kubernetes.io/name=calculadora-backend -n practice-5
kubectl rollout status deployment/calculadora-backend -n practice-5
kubectl exec deployment/calculadora-backend -n practice-5 -- cat /app/logs/api-requests.txt
```

La entrada debe conservarse porque pertenece al PVC, no al sistema de archivos efímero.

## Limpieza

```bash
kubectl delete namespace practice-5
```
