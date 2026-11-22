# Autenticação do dispositivo

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

Só dispositivos cadastrados podem enviar leituras (ver [`requisitos.md`](../requisitos.md), seção 1). A API precisa distinguir um ESP32 legítimo de qualquer outro cliente que tente enviar dados.

## Decisão

- Cada ESP32 carrega no firmware um **código gerador de token**, que produz tokens em um **formato próprio da aplicação**.
- A API só aceita tokens nesse formato. Um dispositivo sem o código gerador não produz um token válido e é rejeitado na autenticação.
- Depois que o token é gerado, o dispositivo é **registrado na aplicação** com o token e um **label**. O label é um nome para identificar o dispositivo e, no futuro, será o nome da casa.
- Leituras de dispositivos não registrados são rejeitadas.

## Consequências

- Nenhum dado entra na API sem passar pela autenticação.
- A segurança depende do sigilo do código gerador: quem conseguir extraí-lo do firmware consegue gerar tokens válidos. Isso torna importante proteger o firmware e fazer o gerador depender de um segredo, e não só do formato do token.
- O label desacopla o dispositivo da casa: a ligação com a casa pode ser feita depois, sem trocar o token.
- Perder ou trocar um ESP32 exige registrar um novo dispositivo.
