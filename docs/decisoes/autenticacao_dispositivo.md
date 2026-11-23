# Autenticação do dispositivo

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

Só dispositivos cadastrados podem enviar leituras (ver [`requisitos.md`](../requisitos.md), seção 1). O ESP32 precisa enviar um identificador próprio e autenticar-se na API sem que o firmware saiba gerar tokens.

## Decisão

- O token é gerado antes do provisionamento por um script administrativo mínimo, fora do ESP32, e gravado no firmware.
- A API guarda apenas o hash do token, usando recursos padrão sempre que possível.
- O script mostra o token uma única vez e registra o dispositivo como ainda sem vínculo a `device_id`.
- O ESP32 envia seu identificador do chip/MAC no campo `device_id` e o token em `Authorization: Bearer <token>`.
- No primeiro uso, a API compara o hash e vincula o token ao `device_id` recebido.
- Depois do vínculo, o mesmo token usado por outro `device_id` é rejeitado.
- O `label` fica para uma versão futura e não participa deste fluxo.
- Revogação ou troca de vínculo não possui tela nesta versão e pode ser feita por processo administrativo futuro.

## Consequências

- O ganho é revogação simples por registro de token e ausência de gerador no firmware.
- O custo é o token ficar gravado no firmware e ser gerado por dispositivo; a proteção do arquivo de configuração e do provisionamento é importante.
- Excluir ou trocar o firmware exige novo provisionamento do token, porque o token não é derivado do hardware.
- O primeiro uso vincula o token de forma definitiva nesta v1; há que evitar reutilizar ou copiar o mesmo token entre equipamentos.
- O hash impede que o banco exponha o token em texto, embora o sigilo ainda dependa do token físico no ESP32.