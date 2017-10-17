# Pthread Lock
## 1.互斥锁
```c
#include <pthread.h>
/* 传入指针，动态分配一个锁 pthread_mutex_t *mutex，所以，不用时需要销毁；也可以静态声明一个 */
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr); 
int pthread_mutex_destroy(pthread_mutex_t *mutex);

/*  静态创建一个锁 */
static pthread_mutex_t lock;

int pthread_mutex_lock(pthread_mutex_t *mutex);		//上锁，失败阻塞
int pthread_mutex_trylock(pthread_mutex_t *mutex);	//尝试上锁，失败不阻塞
int pthread_mutex_unlock(pthread_mutex_t *mutex);	//解锁

```