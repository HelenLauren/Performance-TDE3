# Parte 2: Threads, Semáforos e Condições de Corrida

Nesta segunda parte temos uma demonstração prática de como threads concorrentes podem causar resultados errados ao manipular uma variável compartilhada sem sincronização. O famoso problema da condição de corrida.

Em seguida, temos a resolução do problema utilizando um semáforo binário justo, explorando exclusão mútua, fairness, impacto no desempenho e a garantia de ordem/visibilidade entre threads.


---

### Condição de Corrida no Contador

Quando múltiplas threads executam:

count++;

isso não é uma operação atômica. Internamente a JVM:

-Lê o valor de count.

-Soma 1.

-Escreve o novo valor.

Se duas threads realizam essas etapas ao mesmo tempo uma leitura pode sobrescrever a escrita da outra.
O resultado final perde incrementos (race condition/condição de corrida).


---


# Experimento 1 — Atualização sem controle 

Esse experimento vai mostrar que múltiplas threads incrementando o mesmo contador produzem resultados incorretos.


import java.util.concurrent.*;

public class CorridaSemControle {
    static int count = 0;

    public static void main(String[] args) throws Exception {
        int T = 8, M = 250_000;
        ExecutorService pool = Executors.newFixedThreadPool(T);

        Runnable r = () -> {
            for (int i = 0; i < M; i++) {
                count++; // NÃO ATÔMICO → sujeita a race condition
            }
        };

        long t0 = System.nanoTime();
        for (int i = 0; i < T; i++) pool.submit(r);

        pool.shutdown();
        pool.awaitTermination(1, TimeUnit.MINUTES);
        long t1 = System.nanoTime();

        System.out.printf("Esperado=%d, Obtido=%d, Tempo=%.3fs%n",
                T * M, count, (t1 - t0) / 1e9);
    }
}


*Resultado típico*
Esperado = 2000000
Obtido   = 1.400.000 ~ 1.800.000 (varia)


O contador deveria terminar em 2 milhões.

O valor final é menor, porque várias threads sobrescrevem valores entre si.

O comportamento nunca é igual (nondeterminístico).

Esse código feito demonstra exatamente um cenário com condição de corrida.