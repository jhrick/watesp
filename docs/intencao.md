## 1. Contexto e problema

Várias casas (estilo apartamento) dividem **um único ponto de entrada de água** (registro/hidrômetro geral da companhia de saneamento). Como só existe um medidor, **nenhum morador sabe quanto consome**, e o proprietário não tem como ratear a conta de forma justa nem identificar desperdícios ou vazamentos por unidade.

## 2. Intenção (o "porquê")

Dar **observabilidade do consumo de água por unidade**, de forma automática e acessível de qualquer dispositivo (desktop ou móvel), sem que alguém precise ir olhar um hidrômetro físico.

### Objetivos

1. Medir o consumo de cada residência com um sensor dedicado.
2. Transportar as medições para uma API central de forma confiável.
3. Converter pulsos em litros e disponibilizar histórico e consumo atual.
4. Oferecer duas visões: **Admin** (todas as casas) e **Residente** (somente a sua).
5. Permitir ao admin comparar o consumo do medidor geral com a soma das unidades (detectar perdas/vazamentos nas áreas comuns ou na tubulação).

### Não-objetivos (por enquanto)

- **Não** é um medidor com validade legal/fiscal. O YF-S201 é um sensor de baixo custo; os valores são **estimativas confiáveis para rateio interno**, não substituem o hidrômetro da concessionária.
- Não haverá integração com a companhia de saneamento.
- Não haverá pagamento/cobrança dentro do sistema na v1 (apenas cálculo/relatório de rateio).
- Não haverá controle (abrir/fechar válvula) na v1; apenas leitura.

---

## 3. Usuários e papéis

| Papel | Quem é | O que pode ver/fazer |
|---|---|---|
| `admin` | Dono dos imóveis | Ver todas as unidades, cadastrar/editar unidades, dispositivos e residentes, ver comparativo geral vs. soma, configurar alertas, exportar relatórios |
| `resident` | Morador de uma unidade | Ver **somente** o consumo da própria unidade (atual, histórico, alertas da sua unidade) |

**Regra de ouro de autorização:** um `resident` nunca pode acessar dados de outra unidade, nem por URL direta, nem por ID manipulado. A checagem é feita **no backend**, em toda rota; a UI apenas reflete.

---

## 4. Visão geral da solução

```
┌──────────────┐  pulsos   ┌────────┐  HTTPS (ou MQTT)  ┌──────────────┐     ┌──────────────┐
│ YF-S201      │ ────────► │ ESP32  │ ────────────────► │  API backend │ ──► │  Banco de    │
│ (sensor de   │  (GPIO    │ (firm- │   leituras        │  (regras,    │     │  dados       │
│  fluxo)      │  interr.) │  ware) │                   │  conversão)  │     │  (séries     │
└──────────────┘           └────────┘                   └──────┬───────┘     │  temporais)  │
                                                               │             └──────────────┘
                                                               ▼
                                                    ┌────────────────────┐
                                                    │ Web app responsivo │
                                                    │ (visão admin /     │
                                                    │  visão residente)  │
                                                    └────────────────────┘
```

### Componentes

1. **Firmware (ESP32):** conta pulsos via interrupção, acumula localmente e envia periodicamente.
2. **API backend:** autentica dispositivos e usuários, ingere leituras, converte pulsos em litros, agrega e serve os dados.
3. **Banco de dados:** armazena leituras brutas, cadastro e agregações.
4. **Frontend web responsivo (PWA):** dashboards para admin e residente.
