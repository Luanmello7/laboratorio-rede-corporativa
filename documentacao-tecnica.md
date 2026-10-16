# Documentação técnica — Laboratório de Rede Corporativa N2

## 1. Escopo e limites da verificação

Documento construído com base no histórico da implementação e nos resultados relatados/capturados durante os testes. Não foi possível interpretar o conteúdo binário do arquivo `.pkt` independentemente; por isso, as configurações abaixo são **referência de verificação**, e não exportação automática do projeto.

## 2. Inventário

| Equipamento | Modelo | Conexão / papel |
|---|---|---|
| Router0 | Cisco 2911 | Gi0/0; gateways e DHCP |
| Switch0 | Cisco 2960-24TT | Gi0/1 trunk; Fa0/1–Fa0/4 access |
| PC0 | PC-PT | VLAN 10 — Administração |
| PC1 | PC-PT | VLAN 20 — Atendimento |
| PC2 | PC-PT | VLAN 30 — Suporte TI |
| Server0 | Server-PT | VLAN 30 — DNS e HTTP |

## 3. Rede e VLANs

| VLAN | Nome | Porta(s) | Gateway |
|---|---|---|---|
| 10 | ADM | Fa0/1 | 192.168.10.1/24 |
| 20 | ATENDIMENTO | Fa0/2 | 192.168.20.1/24 |
| 30 | SUPORTE_TI | Fa0/3 e Fa0/4 | 192.168.30.1/24 |

Trunk Gi0/1: encapsulamento 802.1Q; VLANs permitidas 10,20,30. Router-on-a-Stick em Gi0/0.10, Gi0/0.20 e Gi0/0.30.

## 4. Serviços de rede

**DHCP**: três pools (a nomenclatura exata do pool da VLAN 10 foi `VLNA10_ADM`, conforme saída compartilhada). Exclusões dos endereços `.1` a `.9`. Gateways `.1`; DNS `192.168.30.2`. Durante a implantação surgiu um pool adicional `VLAN10_ADM`, depois excluído; o DNS foi associado ao pool original.

**DNS**: Server0 com endereço estático 192.168.30.2; registro A `internet.empresa.luan` → `192.168.30.2`.

**HTTP**: hospedagem local de `index.html` personalizada; acesso verificado em `http://internet.empresa.luan` usando navegador do PC0.

**ACL**: `BLOQUEIO_ATENDIMENTO`, aplicada inbound em Gi0/0.20. Deny 192.168.20.0/24 → 192.168.10.0/24, seguida de permit ip any any. Contadores de correspondência observados durante o teste: 4 pacotes na regra deny, 8 na permit, conforme captura compartilhada.

## 5. Incidentes registrados

### INC-001 — Gateway incorreto

- **Cenário:** gateway do PC1 definido manualmente como 192.168.20.254 (correto: 192.168.20.1).
- **Sintoma:** conectividade local preservada; falha ao acessar servidor 192.168.30.2.
- **Métodos:** `ipconfig`, `ping 192.168.20.1`, `ping 192.168.30.2`.
- **Correção:** alterar gateway para 192.168.20.1.
- **Validação:** conectividade ao servidor restaurada, quatro respostas e perda de 0% conforme captura.
- **Estado final:** PC1 restaurado para DHCP.

### INC-002 — Porta na VLAN incorreta

- **Cenário:** criação temporária da VLAN 99 e mudança da Fa0/2 da VLAN 20 para 99.
- **Sintoma:** PC1 não obteve IP ao renovar DHCP.
- **Métodos:** `show vlan brief`, IP Configuration → DHCP, comparação com o mapeamento de portas.
- **Correção:** `switchport access vlan 20` em Fa0/2 e exclusão da VLAN 99 após o teste.
- **Validação:** nova solicitação DHCP concluída com sucesso, conforme relato do teste.

## 6. Checklist de validação no Packet Tracer

- [ ] `show ip interface brief` no Router0: Gi0/0 e subinterfaces `.10`, `.20`, `.30` em up/up.
- [ ] `show vlan brief` no Switch0: Fa0/1 na VLAN 10; Fa0/2 na 20; Fa0/3 e Fa0/4 na 30; VLAN 99 ausente.
- [ ] `show interfaces trunk` no Switch0: Gi0/1 trunking e VLANs 10,20,30 permitidas.
- [ ] `show running-config` no Router0: três pools corretos com DNS 192.168.30.2.
- [ ] `show access-lists`: ACL cadastrada; `show ip interface gigabitEthernet 0/0.20`: ACL inbound.
- [ ] PCs recebem IP, gateway e DNS esperados por DHCP.
- [ ] PC1 alcança Server0, mas não inicia conexão com VLAN 10.
- [ ] Navegador do PC0 carrega `http://internet.empresa.luan`.
- [ ] Running config salva como startup config nos dois dispositivos; projeto `.pkt` salvo.

## 7. Limitações

O servidor e os roteadores são simulados; não houve teste de desempenho real nem implantação em produção. A filtragem por ACL é stateless e não substitui arquitetura completa de segurança. O nome `.luan` é fictício e usado exclusivamente dentro do laboratório.
