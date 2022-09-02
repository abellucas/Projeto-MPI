# Projeto-MPI
Atividade Avaliativa da disciplina de Sistemas Distribuídos utilizando MPI for Py. Projeto Realizado na plataforma Google Colab.
#Passo a passo para o Funcionamento do Código 
# 1 - Instalação da MPI
`!pip install mpi4py`
# 2 - Código em Python com a MPI4PY
```
# Python
%%writefile mpiobi.py
from mpi4py import MPI

import random
import sys
combinacoes = []
tam_medicoes = int(sys.argv[1])


comm = MPI.COMM_WORLD
id = comm.Get_rank() #id do processo rodando no código
numeroprocessos = comm.Get_size() #número total de processos rodando

inicio = round(id * ((2**tam_medicoes) / numeroprocessos))
fim = round(inicio + ((2**tam_medicoes) / numeroprocessos) -1)

#print(f'Sou o processo {id}, Intervalo: [{inicio}, {fim}]')


for i in range(inicio, fim + 1):
  x = bin(i)
  combinacoes.append(x.split('b')[1].zfill(tam_medicoes))
#print(combinacoes)
print(f'Sou o processo {id}, Intervalo: [{inicio}, {fim}] | {combinacoes}')
print()

soma_usuario = 5
medicao = ''

if id == 0:
    for x in range(0,tam_medicoes):
        medicao += str(random.randint(1,10)) + ' '
    medicao = medicao[:len(medicao)-1].split(' ')
    for i in range(1, numeroprocessos):
      comm.send(medicao, dest = 1, tag = 1)
else:
  medicao = comm.recv(source = 0, tag = 1)
  print(f'Sou o processo {id} e estou recebendo {medicao}\n')

total = 0
for l in combinacoes:
  soma = 0
  fatores = list(l)
  for pos in range(0, len(l)):
    soma += int(fatores[pos]) * int(medicao[pos])
  if soma == soma_usuario:
    total += 1
    print(medicao)
    print(fatores)
    print()
print('Total de combinações com a soma: %d' % total)
print('\n')
```
# 3 - Execução do código com 2 processos e 5 (pode ser qualquer número, mas como exemplo, deixarei 5) passado por parâmetro como o tamanho de medição do problema
`!mpirun --allow-run-as-root -np 2 python mpiobi.py 5`

# 4 - Visualizar o tempo de Execução de 15 a 23 números
```
import time as t
tempo_exec = []
tempo_teorico = []
for i in range(15, 23):
  tempo_atual = t.time()
  string = "mpirun --allow-run-as-root -np 2 python mpiobi.py " + str(i) + " >> /dev/null"
  !$string
  #print(string)
  tempo_final = t.time()
  tempo_final -= tempo_atual
  tempo_exec.append((i, tempo_final))
  tempo_teorico.append((i+1, tempo_final*2))
  print(tempo_final)
print(tempo_exec)
print(tempo_teorico)
```
# 5 - Plotar esses dados de Tempo no Gráfico
```
import matplotlib.pyplot as plt
plt.figure(figsize=(10,7))
plt.plot([tempo_exec[i][0] for i in range(len(tempo_exec))], [tempo_exec[i][1] for i in range(len(tempo_exec))], label='Alg. Paralelo')
plt.plot([tempo_teorico[i][0] for i in range(len(tempo_teorico)-1)], [tempo_teorico[i][1] for i in range(len(tempo_teorico)-1)], label='Previsto')
plt.title('Tempo de execução')
plt.xlabel('Tamanho da Lista')
plt.ylabel('Segundos')
plt.legend()
plt.grid()
plt.show()
```
