# Laboratório de Infraestrutura de Rede Corporativa — Cisco Packet Tracer

Projeto educacional de infraestrutura e suporte técnico N2, realizado no Cisco Packet Tracer. Simula três departamentos e demonstra segmentação lógica, serviços de rede, política de acesso e diagnóstico de incidentes.

> **Escopo e evidências:** as configurações e os testes descritos abaixo foram acompanhados durante a construção do laboratório, por meio de capturas e resultados compartilhados. O arquivo `.pkt` foi recebido, mas não pôde ser inspecionado diretamente fora do Cisco Packet Tracer. A validação final deve ser reproduzida no simulador antes da publicação.

## Arquitetura

- **Router0:** Cisco 2911, conexão `GigabitEthernet0/0` ao switch; subinterfaces 802.1Q `.10`, `.20` e `.30`.
- **Switch0:** Cisco 2960-24TT, uplink `GigabitEthernet0/1` em trunk, VLANs permitidas 10, 20, 30.
- **PC0 (Administração):** porta Fa0/1, VLAN 10.
- **PC1 (Atendimento):** porta Fa0/2, VLAN 20.
- **PC2 (Suporte TI):** porta Fa0/3, VLAN 30.
- **Server0:** porta Fa0/4, VLAN 30, DNS e HTTP.

```text
                 Router0 (2911)
                    Gi0/0
                      |
                802.1Q trunk
                      |
                  Gi0/1
             Switch0 (2960)
        /          |          |          \
     Fa0/1       Fa0/2      Fa0/3       Fa0/4
      PC0          PC1        PC2        Server0
    VLAN 10      VLAN 20    VLAN 30     VLAN 30
```

## Endereçamento IP

| Departamento/dispositivo | VLAN | Rede/endereço | Gateway |
|---|---:|---|---|
| Administração / PC0 | 10 | 192.168.10.0/24 (via DHCP) | 192.168.10.1 |
| Atendimento / PC1 | 20 | 192.168.20.0/24 (via DHCP) | 192.168.20.1 |
| Suporte TI / PC2 | 30 | 192.168.30.0/24 (via DHCP) | 192.168.30.1 |
| Server0 | 30 | 192.168.30.2/24 (estático) | 192.168.30.1 |

Os endereços `.1` a `.9` foram excluídos dos pools DHCP de cada VLAN. Os equipamentos cliente receberam DNS `192.168.30.2` por DHCP após correção de um nome de pool duplicado.

## Funcionalidades implementadas

1. **VLANs e trunk 802.1Q:** segmentação por departamento e transporte das VLANs pelo uplink.
2. **Router-on-a-Stick:** subinterfaces no roteador para comunicação entre VLANs.
3. **DHCP:** pools de atribuição dinâmica para VLANs 10, 20 e 30.
4. **DNS interno:** registro A `internet.empresa.luan` apontando para `192.168.30.2` (nome fictício específico do laboratório, não domínio público registrado).
5. **HTTP:** página de intranet personalizada hospedada no Server0.
6. **ACL estendida:** bloqueio de tráfego iniciado da VLAN 20 para a VLAN 10, preservando o tráfego da VLAN 20 para a VLAN 30.

## Política de acesso

ACL `BLOQUEIO_ATENDIMENTO`, aplicada **inbound** na subinterface `GigabitEthernet0/0.20`:

```cisco
ip access-list extended BLOQUEIO_ATENDIMENTO
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip any any
!
interface GigabitEthernet0/0.20
 ip access-group BLOQUEIO_ATENDIMENTO in
```

**Atenção:** a ACL demonstrada é um filtro de pacotes sem acompanhamento de estado; não equivale a um firewall stateful. O tráfego de resposta também pode ser afetado conforme origem, destino e direção.

## Testes de validação relatados

| Teste | Resultado |
|---|---|
| PC0 alcança gateway VLAN 10 | Sucesso |
| PC1 alcança gateway VLAN 20 | Sucesso |
| PC2 alcança gateway VLAN 30 | Sucesso |
| Comunicação entre os três PCs, antes da ACL | Sucesso |
| PC0 alcança Server0 em 192.168.30.2 | Sucesso |
| Acesso HTTP à intranet pelo IP | Sucesso |
| Resolução DNS e acesso à intranet pelo nome | Sucesso |
| PC1 → VLAN 10 depois da ACL | Bloqueado conforme esperado |
| PC1 → Server0 depois da ACL | Permitido conforme esperado |

## Incidentes simulados

- **INC-001 — Gateway incorreto:** PC1 com gateway 192.168.20.254 podia alcançar dispositivos locais, mas não o servidor 192.168.30.2. Corrigido para 192.168.20.1; conectividade recuperada.
- **INC-002 — VLAN incorreta:** porta Fa0/2 transferida temporariamente para a VLAN 99; PC1 não conseguiu receber IP por DHCP. Porta restaurada para VLAN 20; DHCP recuperado. VLAN 99 removida ao final do exercício.

Detalhes em [documentacao-tecnica.md](documentacao-tecnica.md).

## Como abrir

1. Instale o Cisco Packet Tracer pela Cisco Networking Academy.
2. Abra `Laboratorio_Rede_Corporativa_N2_Final.pkt`.
3. Confira a topologia e o estado das interfaces.
4. Reproduza os testes descritos na documentação.

## Evidências visuais

A pasta `evidencias/` está preparada para capturas da topologia, `show vlan brief`, `show interfaces trunk`, `show ip interface brief`, `show access-lists`, concessões DHCP e página da intranet. **Nenhuma captura foi adicionada automaticamente:** exporte do simulador imagens que não exponham informações reais ou sensíveis.

## Tecnologias

Cisco Packet Tracer · Cisco IOS · VLAN · 802.1Q · TCP/IP · DHCP · DNS · HTTP · ACL · Troubleshooting · Suporte N2

**Natureza:** projeto de simulação educacional, sem alegação de implantação em produção.
