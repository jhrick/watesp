# Tarefas: ingestão de leituras

## API

1. **Criar `api/schema.sql`**
   - Resultado: um único arquivo cria no PostgreSQL dispositivos, tokens por hash, fator por dispositivo e leituras, com unicidade de `(device_id, boot_id, seq)`.
   - Verificação: banco novo é criado pelo arquivo e uma leitura de teste é persistida sem duplicidade.

2. **Criar script administrativo de tokens**
   - Resultado: script gera token seguro, grava apenas o hash e deixa `device_id` não vinculado, mostrando o token uma única vez.
   - Verificação: o banco não contém o token em texto e o script permite um primeiro uso.

3. **Preparar configuração mínima da API**
   - Resultado: `api/app/config.py` lê porta e URL do PostgreSQL por variáveis de ambiente, sem segredos versionados.
   - Verificação: inicialização válida em ambiente sem arquivos secretos no repositório.

4. **Definir e validar o schema da leitura**
   - Resultado: `api/app/schemas/readings.py` valida `device_id`, `pulse_total`, `boot_id` e `seq`, sem `timestamp`.
   - Verificação: campos válidos passam; ausências, tipos incorretos e pulsos negativos são rejeitados com `422`.

5. **Criar regras de autenticação e vínculo**
   - Resultado: hash do token é calculado/comparado com biblioteca padrão; token desconhecido dá `401`, primeiro uso vincula `device_id`, e conflito dá `403`.
   - Verificação: testes cobrem ausente, desconhecido, primeiro uso e troca de `device_id`.

6. **Criar regra de conversão e consumo**
   - Resultado: `api/app/services/ingestion.py` usa 450 pulsos/litro, diferença por `seq` e tratamento de novo `boot_id` como reinício.
   - Verificação: 450 pulsos geram 1 litro; atraso e reinício passam sem consumo negativo.

7. **Implementar persistência de dispositivos e leituras**
   - Resultado: `api/app/repositories/readings.py` persiste pulsos brutos e aplica unicidade da tripla.
   - Verificação: reenvio não cria nova leitura nem novo consumo.

8. **Implementar `POST /readings`**
   - Resultado: `api/app/routes/ingestion.py` processa autenticação, vínculo, idempotência e respostas HTTP síncronas.
   - Verificação: testes cobrem `201`, `200` com `duplicate: true`, `422`, `401`, `403` e `503`.

9. **Adicionar logs da leitura processada**
   - Resultado: logs incluem `device_id`, estado, sequência, pulsos, fator, litros, consumo do intervalo e horário UTC de recebimento.
   - Verificação: um fluxo novo, duplicado e rejeitado deixa registro correto sem token ou segredo.

10. **Adicionar testes de integração da API**
    - Resultado: testes com PostgreSQL cobrem vínculo inicial, conflito de `device_id`, duplicata e atrasados.
    - Verificação: todos os critérios de aceite da API passam.

## ESP32

11. **Definir estrutura mínima e build do firmware**
    - Resultado: `esp32/` possui código principal, dependências próprias e instruções de build.
    - Verificação: o firmware compila em ambiente limpo.

12. **Criar configuração local fora do repositório**
    - Resultado: existe exemplo versionado; arquivo real contém token, identificador obtido do chip, endereço da API e Wi-Fi.
    - Verificação: exemplos não contêm segredo real e o arquivo real não aparece como rastreado.

13. **Implementar contagem de pulsos**
    - Resultado: interrupção YF-S201 incrementa `pulse_total` com segurança e gera novo `boot_id` em cada inicialização.
    - Verificação: testes unitários confirmam incremento e mudança de boot.

14. **Implementar `device_id` e sequenciamento**
    - Resultado: firmware obtém o MAC/chip ID e envia `seq` crescente dentro do mesmo `boot_id`.
    - Verificação: o corpo contém somente `device_id`, `pulse_total`, `boot_id` e `seq`.

15. **Implementar cliente HTTPS**
    - Resultado: firmware faz `POST /readings`, valida certificado e envia o token em `Authorization: Bearer <token>`.
    - Verificação: a API recebe o contrato sem exigir `timestamp` ou litros.

16. **Implementar retenção e reenvio do buffer**
    - Resultado: sem conexão ou com `503`, leituras permanecem no buffer; qualquer `2xx` permite removê-las.
    - Verificação: falha simulada preserva pulsos e sequência e o reenvio não duplica consumo.

17. **Executar teste integrado com hardware**
    - Resultado: ESP32 conta volume conhecido, envia por HTTPS e API persiste pulsos e litros nos logs.
    - Verificação: 450 pulsos produzem aproximadamente 1 litro e reenvio não duplica consumo.