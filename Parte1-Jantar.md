**Relatório — Parte 1: Jantar dos Filósofos**

O Jantar dos Filósofos é um problema clássico usado para explicar dificuldades de coordenação em sistemas concorrentes.
A situação é simples: cinco filósofos sentam ao redor de uma mesa circular. Cada filósofo alterna entre pensar e comer.

Para comer cada filósofo precisa de dois garfos, um que está à sua esquerda e outro à direita. Porém, existe um detalhe importante:

-cada garfo só pode ser usado por um filósofo por vez.

O desafio da atividade é criar um protocolo que permita que todos comam, sem travar o sistema.

---

**Conceitos importantes para resolver o problema:**

1- Exclusão Mútua

Dois filósofos não podem usar o mesmo garfo ao mesmo tempo.
Ou seja, quando um filósofo pega um garfo o outro que compartilha esse garfo deve esperar.

2- Deadlock (Impasse)

O impasse acontece quando todos os filósofos pegam o garfo da esquerda (ou da direita) ao mesmo tempo.

Cada um fica esperando o segundo garfo que nunca ficará livre, porque todos estão segurando um garfo e esperando eternamente pelo outro.

No final ninguém come e ninguém devolve o garfo ai o sistema trava.

3- Fome (Starvation)

Mesmo que não aconteca o deadlock, um filósofo pode ficar muito tempo sem conseguir pegar os dois garfos e acabar ficando sem comer.
Ai temos o segundo problema: fome.

---


Para resolver, o protocolo mais simples seria:

1.O filósofo tenta pegar o garfo da esquerda. Depois tenta pegar o garfo da direita.

2.Come. 

3.Devolve os dois garfos.

Esse protocolo apesar de lógico e direto, tem um problema.

Todos podem pegar o primeiro garfo ao mesmo tempo, e quando forem tentar pegar o segundo garfo, ninguém consegue, o sistema trava. Criando um ciclo de espera:

F 1 espera o garfo do F 2

F= 2 espera o garfo do F 3

F 3 espera o garfo do F 4

F 4 espera o garfo do F 5

F 5 espera o garfo do F 1

F=Filósofo

Ou seja, existe espera circular. Uma das condições clássicas para deadlocks...

Para que não ocorra essa deadlock, temos que impedir essa espera circular. Então devemos impor uma **ordem fixa**!!

---


**A maneira mais lógica de resolver é, por exemplo:**

-Filósofos com número ímpar pegam primeiro o garfo da direita e depois o da esquerda.

-Filósofos com número par pegam primeiro o garfo da esquerda e depois o da direita.

Assim, pelo menos um filósofo consegue pegar os dois garfos e comer, liberando os garfos para os próximos na ordem.
O ciclo de espera deixa de existir porque a ordem de aquisição não é mais a mesma para todos...

Essa solução básica e muito famosa:

-Quebra a "espera circular" ao impedir que todos adquiram garfos na mesma ordem.

-Garante que todos irão comer e pensar, pois filósofo sempre conseguirá adquirir os dois garfos e liberar depois.

-Evita o travamento TOTAL, pois nenhum filósofo vai segurar os dois garfos ao mesmo tempo.

---

**Código lógico que resolve o problema do JANTAR DOS FILÓSOFOS:**

obs: de acordo com a lógica explicada anteriormente :)


 número de filósofos e garfos: **N = 5**

cada garfo é um recurso exclusivo: **garfo[0..N-1]**

para cada filósofo f de 0 até N-1:

    garfo_esquerda = f
    garfo_direita  = (f + 1) mod N

    criar thread para o filósofo f executando:

        enquanto verdadeiro:
            pensar()
            estado[f] = "com fome"

            // ordem depende se o filósofo é par ou ímpar
            se Filósofo é PAR então
                adquirir(garfo_esquerda)
                adquirir(garfo_direita)
            senão                           // Filósofo é ÍMPAR
                adquirir(garfo_direita)
                adquirir(garfo_esquerda)

            estado[f] = "comendo"
            comer()

            // liberar sempre os dois garfos
            liberar(garfo_esquerda)
            liberar(garfo_direita)

            estado[f] = "pensando"
