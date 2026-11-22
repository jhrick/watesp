**Versão:** v1.0

**Princípio do MVP:** mostrar o consumo de água de cada casa em uma interface simples. Tudo o que não ajuda nisso fica para depois.

---

## 1. Ingestão de leituras

O sistema recebe as medições enviadas pelo sensor de cada casa e as guarda de forma confiável.

**O que precisa acontecer**

- Cada casa tem um dispositivo identificado, e só dispositivos cadastrados podem enviar leituras.
- O dispositivo envia a quantidade de **pulsos** medida; a conversão para **litros** é feita pelo sistema.
- Os pulsos brutos são sempre guardados. O litro é calculado a partir deles, para que seja possível recalibrar depois.
- Reenviar a mesma leitura não pode contar o consumo duas vezes.
- Se o dispositivo ficar sem conexão e enviar leituras atrasadas, elas devem ser aceitas e contadas no período em que ocorreram.
- Se o dispositivo reiniciar, o consumo não pode ficar negativo nem dar um salto irreal.

**Pronto quando**

- [ ] Um dispositivo de teste envia leituras e elas aparecem armazenadas, sem duplicatas.
- [ ] Leituras atrasadas (simulando queda de conexão) são aceitas sem perda.
- [ ] Um reinício simulado do dispositivo não gera consumo negativo.
- [ ] Um volume conhecido (ex.: 10 L) resulta em um valor em litros próximo do real.

---

## 2. Autenticação e acesso aos dados

Cada pessoa vê somente o que lhe cabe.

**O que precisa acontecer**

- Usuários entram com e-mail e senha.
- Existem dois perfis:
  - **Admin:** vê todas as casas.
  - **Residente:** vê apenas a própria casa.
- Um residente não consegue acessar dados de outra casa, nem alterando o endereço ou o identificador na requisição. A regra vale no servidor, não só na tela.
- Na v1, o cadastro de casas, dispositivos e moradores é feito pelo admin de forma simples (pode ser manual, sem tela própria).

**Pronto quando**

- [ ] Login e logout funcionam.
- [ ] Residente só enxerga a própria casa; tentar acessar outra é bloqueado.
- [ ] Admin enxerga todas as casas.

---

## 3. Interface

Uma tela fácil de entender, acessível pelo celular ou pelo computador.

**Residente**

- Consumo de hoje e do mês atual, em litros.
- Gráfico do consumo ao longo do tempo (por dia no mês, por hora no dia).

**Admin**

- Lista de todas as casas com o consumo do dia e do mês.
- Ao escolher uma casa, vê a mesma tela que o residente vê.

**Geral**

- Funciona bem em tela de celular.
- Mostra mensagens claras quando não há dados ou quando o sensor está sem enviar leituras.

**Pronto quando**

- [ ] Um residente abre o app no celular e entende quanto consumiu hoje e no mês.
- [ ] O admin consegue comparar as casas em uma única tela.

