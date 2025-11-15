# Parte 3 — Deadlock em Sistemas Concorrentes

Esta parte demonstra como ocorre um **deadlock** em Java quando duas threads tentam adquirir dois recursos (locks) em ordens opostas.  
Também é apresentada a correção através de **hierarquia de recursos**, técnica que elimina a condição de espera circular.

---

#  1. O que é Deadlock?

Deadlock ocorre quando duas ou mais threads ficam **paradas para sempre**, porque cada uma está esperando por um recurso que nunca será liberado.

No exemplo implementado:

- Thread 1 pega **LOCK_A** e quer **LOCK_B**
- Thread 2 pega **LOCK_B** e quer **LOCK_A**

Nenhuma libera o lock que está segurando → o programa **trava**.

---

# 2. Código que produz Deadlock

```java
public class DeadlockDemo {

    static final Object LOCK_A = new Object();
    static final Object LOCK_B = new Object();

    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            System.out.println("T1: tentando pegar A...");
            synchronized (LOCK_A) {
                System.out.println("T1: segurando A...");
                dormir(80);

                System.out.println("T1: tentando pegar B...");
                synchronized (LOCK_B) {
                    System.out.println("T1 terminou.");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            System.out.println("T2: tentando pegar B...");
            synchronized (LOCK_B) {
                System.out.println("T2: segurando B...");
                dormir(80);

                System.out.println("T2: tentando pegar A...");
                synchronized (LOCK_A) {
                    System.out.println("T2 terminou.");
                }
            }
        });

        t1.start();
        t2.start();
    }

    static void dormir(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```



#  3. Logs da Execução (prova do Deadlock)

    T1: tentando pegar A...
    T1: segurando A...
    T2: tentando pegar B...
    T2: segurando B...
    T2: tentando pegar A...
    T1: tentando pegar B...

    Depois disso: o programa congela.
    Não existe exceção, não existe erro — apenas o deadlock.

# 4. Condições de Coffman (todas presentes)

    Condição	O que significa	Como aparece no código
    Exclusão Mútua	Um recurso só pode ser usado por uma thread	synchronized (LOCK_A/B)
    Manter e Esperar	Segura um recurso enquanto espera outro	T1 segura A e espera B; T2 segura B e espera A
    Não Preempção	Recurso não pode ser tomado à força	Locks só são liberados voluntariamente
    Espera Circular	Um espera pelo outro em um ciclo	A → B → A (T1 espera T2 e vice-versa)

    Como as quatro condições ocorrem juntas, o deadlock é inevitável.

# 5. Código Corrigido (sem Deadlock)

    Solução: hierarquia de recursos
    → Todas as threads devem pegar sempre na ordem: A → B

```java

    public class DeadlockFixedDemo {

        static final Object LOCK_A = new Object();
        static final Object LOCK_B = new Object();

        public static void main(String[] args) {

            Thread t1 = new Thread(() -> {
                System.out.println("T1: tentando pegar A...");
                synchronized (LOCK_A) {
                    System.out.println("T1: segurando A...");
                    dormir(70);

                    System.out.println("T1: tentando pegar B...");
                    synchronized (LOCK_B) {
                        System.out.println("T1 terminou com sucesso.");
                    }
                }
            });

            Thread t2 = new Thread(() -> {
                System.out.println("T2: tentando pegar A...");
                synchronized (LOCK_A) {
                    System.out.println("T2: segurando A...");
                    dormir(70);

                    System.out.println("T2: tentando pegar B...");
                    synchronized (LOCK_B) {
                        System.out.println("T2 terminou com sucesso.");
                    }
                }
            });

            t1.start();
            t2.start();
        }

        static void dormir(long ms) {
            try { Thread.sleep(ms); } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

```


# 6. Por que funciona?

    Porque quebra a condição de Espera Circular, uma das quatro condições de Coffman.

    Continuação da exclusão mútua ✔

    Continua manter-e-esperar ✔

    Continua não-preempção ✔

    Não há mais ciclo (A → B → A)

    Sem espera circular → não existe possibilidade de deadlock.



# 7. Conexão com o Jantar dos Filósofos

    Essa é a mesma estratégia usada no problema dos filósofos:

    Definir uma ordem fixa para pegar os garfos

    Evita que todos peguem o primeiro garfo ao mesmo tempo

    Elimina a espera circular

    É a estratégia mais simples, mais barata e mais usada em sistemas reais


# Conclusão

    A Parte 3 demonstra:

    deadlock real

    mapeamento das 4 condições de Coffman

    uma solução prática e eficiente (hierarquia de recursos)

    relação com conceitos clássicos de concorrência