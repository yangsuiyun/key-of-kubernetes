1. owner

   * 创建

     ```go
     metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
     
     OwnerReferences: []metav1.OwnerReference{
        *metav1.NewControllerRef(foo, samplev1alpha1.SchemeGroupVersion.WithKind("AppCtl")),
     }
     ```

   * 获取

     ```go
     ownerRef := metav1.GetControllerOf(object)
     ```

   * 判断

     ```go
     metav1.IsControlledBy(deployment, app)
     ```

