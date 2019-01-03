# Resumos SOPE
Notas para exame sobre toda a matéria.
## Teoria dos Sistemas Operativos
### Introdução
### Hardware de um sistema de computação
### Estrutura de um sistema operativo
### Processos e threads
### Escalonamento do processador
### Sincronização de processos
### Deadlocks
### Gestão de memória
### Memória virtual
### Sistema de ficheiros

## API do Unix
### Aspetos gerais dos programas em C no UNIX
#### Terminação de um programa
```c
void exit(int status);
```
#### Tratamento de erros

```c
void perror (const char *msg);
```
Mostra msg seguido de ':', e descrição do último erro em chamada ao sistema.
### Consola, Ficheiros e Diretórios
#### Descritores
Constantes:
STDIN_FILENO, STDOUT_FILENO,  STDERR_FILENO

#### Criação/abertura de ficheiros
```c
int open(const char *pathname, int oflag, ... /*, mode_t mode */);
```
Retorno: descritor do ficheiro se OK, -1 se erro
oflag: combinação das seguintes flags:

 - O_RDONLY
 - O_WRONLY
 - O_RDWR
 - O_APPEND
 - O_CREAT - criar ficheiro se não existir, requer mode
 - O_EXCL - origina erro se ficheiro existir e O_CREAT ativado
 - O_TRUNC - ficheiro fica com comprimento 0

mode: permissões do ficheiro, em octal

#### Duplicação de um descritor

```c
int dup (int filedes);
```
Retorno: novo descritor se OK, -1 se erro
Procura descritor livre mais baixo e põe-no a apontar para o mesmo ficheiro que filedes.

```c
int dup2 (int filedes, int filedes2);
```
Retorno: novo descritor se OK, -1 se erro
Fecha filedes2 se estiver aberto, e põe-no a apontar para o mesmo ficheiro que filedes.

#### Leitura de um ficheiro

```c
ssize_t read(int filedes, void *buff, size_t nbytes);
```
Retorna: o nº de bytes lidos, 0 se fim do ficheiro, -1 se erro

#### Escrita de um ficheiro

```c
ssize_t write(int filedes, const void *buff, size_t nbytes);
```
Retorna: o nº de bytes escritos, -1 se erro


#### Fecho de um ficheiro

```c
int close(int filedes);
```
Retorna: 0 se OK, -1 se erro.

#### Apagamento de um ficheiro
```c
int unlink(const char *pathname);
```
Retorna: 0 se OK, -1 se erro.
### Criação e Terminação de Processos
#### A função fork
```c
pid_t fork(void);
```
Retorna: 0 para filho, PID do filho para o pai, -1 se erro.
Após chamada, ambos os processos executam o mesmo código (usar ifs com o valor de retorno).
Após fork, não se sabe quem começa a executar primeiro.
Filho fica com cópia do segmento de dados, *heap*, e *stack*.
O processo filho pode aceder e alterar as variáveis declaradas antes de fork().
```c
pid_t getpid(void); /* Obter a PID do próprio processo */
pid_t getppid(void); /* Obter a PID do processo-pai */
```
#### Processos zombie
Processo que terminou mas cujo pai ainda não executou um `wait`.

#### As funções wait e waitpid
Usando estas funções, o pai espera pela terminação do filho, aceitando o seu código.
Quando processo termina, é enviado ao pai o sinal SIGCHLD.
```c
pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
```
Retorno: PID do processo se OK, -1 se erro.

#### Macros para testar exit status do filho

 - WIFEXITED(status)
	 valor positivo, se terminou normalmente
	 WEXITSTATUS(status) permite obter exit status do filho
 - WIFSIGNALED(status)
	 valor positivo, se recebeu sinal que não tratou
 - WIFSTOPPED(status)
	valor positivo se filho está parado

#### As funções exec
```c
int execl (const char *pathname, const char *arg0, ... /* (char *)0 */);
int execv (const char *pathname, char *const argv[]);
int execle(const char *pathname, const char *arg0, ... /* (char *)0,
char *const envp[] */);
int execve(const char *pathname, char *const argv[], char *const envp[]);
int execlp(const char *filename, const char *arg0, ... /* (char *)0 */);
int execvp(const char *filename, char *const argv[]);
```
Apenas retornam se ocorrer erro (-1).

 - l
argumentos passados um a um, terminados por NULL
 - v
argumentos passados em array
 - e
passa-se variáveis de ambiente
 - p
filename é nome do ficheiro executável

### Sinais
#### Tratamento dos sinais após fork / exec
Após fork:
Tratamento dos sinais é herdado, mas pode ser alterado.

Após exec:
Tratamento é por omissão, ou ignorar se o processo que invocou exec também estiver a ignorar.

#### As funções kill e raise
```c
int kill (pid_t pid, int signo);
```
Retorno: 0 se OK, -1 se erro
Envia um sinal ao processo pid.

#### As funções alarm e pause
```c
unsigned int alarm(unsigned int count);
```
Retorno: 0 se OK; -1 se ocorreu erro
Processo que invocou recebe  SIGALRM após count segundos.
Se count = 0, algum alarme pendente é cancelado.

#### Funções Posix p/sinais
```c
int sigaction(int signum, const struct sigaction *action,
struct sigaction *oldaction);
```
Retorno: 0 se OK; -1 se ocorreu erro

Exemplo de instalação:
```c
struct sigaction action;
action.sa_handler = sigint_handler;
sigemptyset(&action.sa_mask);
action.sa_flags = 0;
sigaction(SIGINT,&action,NULL);
```
### Pipes e FIFOS

#### Pipes vs FIFOS
Em ambos, os dados apenas podem fluir num sentido.
Os pipes têm que ser usados por processos com antecessor comum, enquanto que no caso dos fifos estes não têm que ser relacionados.

#### Pipes
Funcionalidade:

 - Criar pipe;
 - Invocar fork;
 - Escritor fecha extremidade de leitura, leitor fecha de escrita.   
- Usar Write e read;  
- No final, fechar outras extremidades

fd[0] -> aberto para leitura
fd[1] -> aberto para escrita
```c
int pipe (int fd[2]);
```
Retorno: 0 se ok, -1 se erro

  
```c
FILE *popen(const char *cmdstring, const char *type);
```
cmdstring -> programa a executar
type -> r para leitura, w para escrita

Retorna: file pointer se OK; NULL se houve erro
```c
int pclose(FILE *fp);
```
Retorna: termination status de cmdstring se OK; -1 se houve erro
#### FIFOS
Tipo de ficheiro. Tem um nome e pode ser visto no sistema de ficheiros.
No terminal existem os seguintes utilitários:

`mkfifo nome` -> criar fifo com nome

`rm nome` -> elimina fifo com nome

A API fornece as seguintes funções:

```c
int mkfifo(const char *pathname, mode_t mode);
```

Retorna EEXIST se fifo já existir.  
Retorno: 0 se OK, -1 se erro

Onde mode representa as permissões de acesso, em octal. Começa por 0, e os três restantes algarismos representam as permissões de leitura, escrita e execução, para o dono, grupo, outros, respetivamente.

Para manipular o fifo, usar as funções já referidas `open`, `close`, `read`, e `write`.

##### Regras:

Abertura:
em modo O_RDONLY, espera que outra extremidade seja aberta para escrita, se não estiver e O_NONBLOCK estiver desativada.
em modo O_WRONLY, espera que outra extremidade seja aberta para escrita, se não estiver e O_NONBLOCK estiver desativada. Se estiver ativada, retorna -1.

Leitura/Escrita
Escrita para FIFO que não está aberto para leitura - SIGPIPE
  
 ```c
int unlink (const char *pathname);
 ```
Destrói o fifo de pathname.
### Threads
#### Criação de threads
  ```c
int pthread_create ( pthread_t *tid, const pthread_attr_t *attr, void * (*func)(void *), void *arg );
 ```

 - tid
	 apontador para identificador do thread (preenchido pela função) 
 - attr
    atributos do thread, normalmento NULL (valor por defeito)
 - func
	 função executada pelo thread
 - arg
	 apontador para argumento do thread
	 
####  Terminação de threads
Quando main termina, threads criados por si também são terminados.
Mas, se terminar com chamada `pthread_exit()`, os outros threads continuam em execução.
 ```c
void pthread_exit (void *status);
 ```
Não retorna.

  ```c
int pthread_join (pthread_t tid, void **status);
 ```
 Thread bloqueia até que thread tid terminar.
#### Dicas para uso de threads
Função da thread pode retornar qualquer apontador. Não esquecer de fazer `free`no final do programa.
Passagem de argumentos:
Ao criar threads em ciclo for, os argumentos devem ser passados através de um array externo: 
Errado:
 ```c
for(t=0; t<NUM_THREADS; t++){
printf("Creating thread %d\n", t);
pthread_create(&tid[t], NULL, PrintHello, &t);
}
```
o ciclo que cria os threads modifica
o conteúdo do endereço passado como argumento
possivelmente antes de o thread criado conseguir aceder-lhe

Certo:
```c
int thrarg[NUM_THREADS];

for(t=0;t < NUM_THREADS;t++)
{
	thrarg[t] = t;
	printf("Creating thread %d\n", t);
	pthread_create(&threads[t], NULL,
	PrintHello,
	&thrarg[t]);
	...
}
```
Para passar vários argumentos, usar structs.

Saber o tid da própria thread:
 ```c
pthread_t pthread_self (void);
 ```




### Filas de Mensagens, Memória Partilhada, Semáforos, Mutexes, Condition variables

#### Comunicação entre Processos
Correndo na mesma máquina:
 - Pipes e FIFOS
 - Mensagens
 - Semáforos
 - Memória partilhada

#### Semáforos
semáforos com nome podem ser partilhados por vários processos, sem nome por processos com acesso a memória comum
```c
sem_t* sem_open(char* name, int flags, mode_t mode, unsigned value);
int sem_unlink(char* name);
int sem_init(sem_t* sem, int pshared, unsigned value);
int sem_close(sem_t* sem);
int sem_destroy(sem_t* sem);
int sem_getvalue(sem_t* sem, int* sval);
int sem_wait(sem_t* sem);
int sem_trywait(sem_t* sem);
int sem_post(sem_t* sem);
```
name deve ter '/' no início.


#### Memória partilhada

#### Sincronização de threads

##### Mutexes
Inicialização
```c
pthread_mutex_t mymutex = PTHREAD_MUTEX_INITIALIZER;
/* ou */
int pthread_mutex_init ( pthread_mutex_t *mutx,
const pthread_mutexattr_t *attr);

Lock e unlock
```
```c
int pthread_mutex_lock (pthread_mutex_t *mutx);
/* tenta adquirir o mutex, se já estiver locked, bloqueia até que fique unlocked */
int pthread_mutex_trylock (pthread_mutex_t *mutx);
/* se estiver unlocked, faz lock, senão
retorna EBUSY */
int pthread_mutex_unlock (pthread_mutex_t *mutx);
/* faz unlock do mutex, retorna erro se já estiver unlocked ou estiver na posse de outro thread */
int pthread_mutex_destroy (pthread_mutex_t *mutx);
/* destrói o mutex */
```
##### Condition variables
Inicialização:
```c
pthread_cond_t mycondvar = PTHREAD_COND_INITIALIZER;
/* ou */
int pthread_cond_init ( pthread_cond_t *cvar,
const pthread_condattr_t *attr);
```

```c
/*
bloqueia thread até condição se assinalar
deve ser chamada após pthread_mutex_lock()
*/
int pthread_cond_wait (pthread_cond_t *cvar, pthread_mutex_t *mutx);

/*
assinala/acorda outro thread
*/
int pthread_cond_signal (pthread_cond_t *cvar);
/*
desbloqueia todos threads que estiverem bloqueados em cvar
*/
int pthread_cond_broadcast (pthread_cond_t *cvar);

/*
destrói condition variable
*/
int pthread_cond_destroy (pthread_cond_t *cvar);


```

## Funções de C
