## 10. Vendas: Validação para evitar pedido duplicado

### 🛠 Problema real
Em muitas empresas, o time comercial ou de atendimento acaba **lançando pedidos duplicados** no Protheus (SC5/SC6) por acidente — seja por erro na digitação, falha de comunicação ou lentidão no sistema.

Esses pedidos geram **duplicidade no faturamento, estoque reservado em dobro e cobranças indevidas ao cliente**.

### 📉 Impacto
- Faturamento em duplicidade
- Cobrança incorreta ao cliente
- Estoque baixado desnecessariamente
- Desconfiança do cliente e retrabalho
- Necessidade de estorno, devolução ou cancelamento

### 💡 Solução aplicada
Criação de uma **validação automática no momento da inclusão do pedido** para verificar se já existe um pedido **com as mesmas condições**: cliente, produto, quantidade e data próxima.

Se identificado um possível pedido duplicado, o sistema exibe um alerta e bloqueia ou solicita confirmação do usuário.

### 🧾 Código exemplo (ADVPL)
```advpl
User Function ValidDup()
    Local cCliente := SC5->C5_CLIENTE
    Local dHoje := dDataBase
    Local lDuplicado := .F.

    dbSelectArea("SC5")
    dbSetOrder(1) // Índice por cliente
    dbSeek(xFilial("SC5") + cCliente)

    While !Eof() .And. SC5->C5_CLIENTE == cCliente
        If Abs(SC5->C5_EMISSAO - dHoje) <= 3 .And. SC5->C5_PRODUTO == SC6->C6_PRODUTO
            lDuplicado := .T.
            Exit
        EndIf
        dbSkip()
    EndDo

    If lDuplicado
        MsgStop("Existe um pedido semelhante lançado recentemente para este cliente.")
        Return .F.
    EndIf

Return .T.
```
> A regra pode ser ajustada com base em critérios específicos da empresa (ex: quantidade, prazo, valor mínimo, etc.).

## 🧪 Casos de Teste - Rotina de Validação SC5/SC6

| Cenário | Campos Verificados | Regra | Ação | Código Erro |
|---------|--------------------|-------|------|-------------|
| Duplicado exato | C5_CLIENTE, C5_PRODUTO, C5_DATA | C5_CONFIRMADO = "N" | ❌ Bloqueio | PD-409 |
| Variação de produto | C5_PRODUTO ≠ anterior | - | ✅ Libera | - |
| Novo cliente | C5_CLIENTE ≠ anterior | - | ✅ Libera | - |

**Parâmetros:**
- Margem de datas: 7 dias para "data próxima"
- Tolerância: 3 alertas/dia por cliente

**Fluxo de Decisão:**
1. Sistema compara:
   - ✅ Cliente diferente → Libera
   - ✅ Produto diferente → Libera
   - ❌ Tudo igual → Bloqueia

**Exceções:**
- Pedidos com flag "RECORRENTE" ignoram validação
- Vendedores Nível 3 podem contornar alertas

**Melhorias Planejadas:**
> - Adicionar verificação por similaridade (fuzzy matching)
> - Criar whitelist de produtos isentos
> - Integrar com histórico de compras

🎯 Benefícios
Redução de retrabalho e devoluções

Evita prejuízos com faturamento duplicado

Maior controle de pedidos ativos

Mais segurança no processo comercial

🏷️ Tags
#Protheus #SC5 #SC6 #Vendas #Duplicidade #Validação #ADVPL #Pedidos
