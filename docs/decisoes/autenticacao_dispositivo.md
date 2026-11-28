# Autenticação do dispositivo

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

Só dispositivos cadastrados podem enviar leituras (ver [`requisitos.md`](../requisitos.md), seção 1). O ESP32 precisa enviar um identificador próprio e autenticar-se na API sem que o firmware saiba gerar tokens.

## Decisão

- O token é gerado antes do provisionamento por um script administrativo mínimo, fora do ESP32, e gravado no firmware.
- No momento da geração, o script grava no banco apenas o hash do token e o mostra em texto uma única vez; o registro fica inicialmente sem vínculo a `device_id`.
- Ao receber uma autenticação, a API calcula o hash do token enviado e verifica se ele corresponde a um hash já registrado no banco. Só então registra o uso do token ou realiza o vínculo.
- O ESP32 envia seu identificador do chip/MAC no campo `device_id` e o token em `Authorization: Bearer <token>`.
- No primeiro uso válido, a API grava o `device_id` recebido e vincula o token a esse dispositivo.
- Depois do vínculo, o mesmo token usado por outro `device_id` é rejeitado.
- O `label` fica para uma versão futura e não participa deste fluxo.
- Revogação ou troca de vínculo não possui tela nesta versão e pode ser feita por processo administrativo futuro.

## Consequências

- O ganho é revogação simples por registro de token e ausência de gerador no firmware.
- O registro prévio do hash permite verificar que o token foi gerado pela aplicação; sem essa verificação, qualquer token enviado poderia ser aceito e gravado no banco.
- O custo é o token ficar gravado no firmware e ser gerado por dispositivo; a proteção do arquivo de configuração e do provisionamento é importante.
- Excluir ou trocar o firmware exige novo provisionamento do token, porque o token não é derivado do hardware.
- O primeiro uso vincula o token de forma definitiva nesta v1; há que evitar reutilizar ou copiar o mesmo token entre equipamentos.
- O hash impede que o banco exponha o token em texto, embora o sigilo ainda dependa do token físico no ESP32.
