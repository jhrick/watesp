# Transporte entre dispositivo e API

- **Estado:** aceita
- **Data:** 2026-11-22

## Contexto

Cada ESP32 envia suas leituras para a API pela rede Wi-Fi da casa. O envio é periódico e não precisa ser em tempo real.

## Decisão

O dispositivo envia as leituras à API por **HTTPS**.

## Consequências

- O ESP32 e a API usam o mesmo protocolo que o resto da aplicação, sem broker ou infraestrutura adicional.
- O tráfego é cifrado, o que protege o token do dispositivo e os dados de consumo.
- O dispositivo precisa conseguir validar o certificado do servidor.
- Se um dia for necessária comunicação em tempo real ou do servidor para o dispositivo, esta decisão deve ser revista.
