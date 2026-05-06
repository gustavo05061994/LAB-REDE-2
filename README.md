# Projeto CCNA 200-301 — Rede Corporativa Multi-Site no Cisco Packet Tracer

Segundo projeto acumulativo de preparação para a CCNA 200-301, desenvolvido no Cisco Packet Tracer com metodologia incremental — cada lab adiciona complexidade ao anterior, resultando em uma rede corporativa completa construída do zero.

---

## Sobre o projeto

A proposta foi expandir a topologia do primeiro projeto e simular uma rede corporativa real com matriz e filiais. Quatro departamentos interconectados em cascata, cada um com sua VLAN, dispositivos, regras de acesso e serviços — tudo precisando funcionar junto.

Ao final do projeto, a rede conta com roteamento inter-VLAN, DHCP centralizado, telefonia IP, redes wireless, segurança por ACLs, NAT para saída à internet e redundância de camada 2 com EtherChannel e RSTP.

---

## Topologia final

```
Internet (ISP)
      |
   R1 (2911) ─────────────────────── R2 (2911)
   Gig0/0(sub-interfaces)           Gig0/0 (10.0.0.6)
   Gig0/1 (10.0.0.5)                Gig0/1 (172.16.0.1)
   Gig0/2 (200.1.1.1 - NAT outside) Gig0/2 (172.17.0.1)
      |
   SW-TI (3560) ══════════════ SW-GERENCIA (2960)
   Gig0/1 → R1                 Po1 (EtherChannel LACP)
   Gig0/2 → R2                 Gig0/1 → R1
   Fa0/15-16 → Po1             Gig0/2 → SW-RH
      |                              |
   Dispositivos TI              SW-RH (2960)
   PC0, PC1, PC2                Gig0/1 → SW-GERENCIA
   IMPRESSORA-TI                Gig0/2 → SW-ENGENHARIA
   AP-TI, Laptop0                    |
   IP Phone0                    Dispositivos RH
   WLC-PT                       PC7~PC14, Printer2
                                 AP-RH, Laptop1
                                      |
                                 SW-ENGENHARIA (2960)
                                 Gig0/2 → SW-RH
                                      |
                                 Dispositivos Engenharia
                                 PC15~PC24, Printer3
                                 AP-ENGENHARIA, Laptop2
```

### Equipamentos
- 2x Roteador Cisco 2911
- 1x Switch Cisco 3560 (SW-TI — Layer 3)
- 3x Switch Cisco 2960 (SW-GERENCIA, SW-RH, SW-ENGENHARIA)
- 25x PCs
- 3x Impressoras
- 4x Access Points
- 4x Notebooks wireless
- 1x IP Phone
- 1x WLC-PT

### Endereçamento IP

| Dispositivo | Interface | IP | Descrição |
|---|---|---|---|
| R1 | Gig0/0.10 | 192.168.10.1/24 | Gateway VLAN 10 (TI/VoIP) |
| R1 | Gig0/0.20 | 192.168.20.1/24 | Gateway VLAN 20 (Gerência) |
| R1 | Gig0/0.30 | 192.168.30.1/24 | Gateway VLAN 30 (RH) |
| R1 | Gig0/0.40 | 192.168.40.1/24 | Gateway VLAN 40 (Engenharia) |
| R1 | Gig0/1 | 10.0.0.5/30 | Link para SW-TI |
| R1 | Gig0/2 | 200.1.1.1/30 | Saída para internet (NAT outside) |
| R2 | Gig0/0 | 10.0.0.6/30 | Link para SW-TI |
| WLC-PT | Management | 192.168.10.10/24 | Controladora wireless |
| IMPRESSORA-TI | Fa0 | 192.168.10.10 (estático) | Fixa — excluída do pool |
| Printer2 (RH) | Fa0 | 192.168.30.10 (estático) | Fixa — excluída do pool |
| Printer3 (ENG) | Fa0 | 192.168.40.10 (estático) | Fixa — excluída do pool |

### VLANs

| VLAN | Nome | Rede | Dispositivos |
|---|---|---|---|
| 10 | TI / VoIP | 192.168.10.0/24 | PCs TI, IP Phone, AP-TI, Impressora |
| 20 | Gerência | 192.168.20.0/24 | PCs Gerência, AP-GERENCIA, Impressora |
| 30 | RH | 192.168.30.0/24 | PCs RH, AP-RH, Impressora |
| 40 | Engenharia | 192.168.40.0/24 | PCs Engenharia, AP-ENGENHARIA, Impressora |
| 99 | Roteadores | 10.0.0.4/30 | Link R1 ↔ R2 via SW-TI |

---

## Labs

### Lab 1 — Configurações básicas
Configuração inicial de todos os dispositivos: hostname, senhas, banner de aviso e acesso seguro via console e VTY. Base de segurança aplicada antes de qualquer configuração de rede.

**Comandos principais:** `hostname`, `enable secret`, `banner motd`, `line con 0`, `line vty 0 4`, `login`, `write memory`

---

### Lab 2 — VLANs e roteamento inter-VLAN
Criação das VLANs 10, 20, 30 e 40 em todos os switches. Configuração de portas de acesso por dispositivo e trunks entre switches. Router-on-a-Stick no R1 com sub-interfaces e encapsulamento dot1Q para rotear entre as VLANs.

**Comandos principais:** `vlan 10`, `switchport mode access`, `switchport access vlan 10`, `switchport mode trunk`, `interface gig0/0.10`, `encapsulation dot1q 10`, `show vlan brief`, `show interfaces trunk`

---

### Lab 3 — Roteamento dinâmico (OSPF)
Configuração do OSPF área 0 no R1, R2 e SW-TI para troca dinâmica de rotas entre os equipamentos. Verificação de adjacências e tabela de roteamento.

**Comandos principais:** `router ospf 1`, `router-id`, `network x.x.x.x wildcard area 0`, `show ip ospf neighbor`, `show ip route`

---

### Lab 4 — DHCP e Telefonia IP
DHCP centralizado no R1 com pools separados por VLAN. Endereços excluídos para gateways e dispositivos fixos. Telephony-service configurado para VoIP com ephone-dn, registro do IP Phone e DHCP opção 150. APs configurados com WPA2-PSK e notebooks conectados via wireless.

**Comandos principais:** `ip dhcp excluded-address`, `ip dhcp pool`, `default-router`, `dns-server`, `option 150 ip`, `telephony-service`, `ip source-address`, `ephone-dn`, `ephone`, `button 1:1`, `show ip dhcp binding`

---

### Lab 5 — STP e EtherChannel avançado
EtherChannel LACP (Po1) entre SW-TI e SW-GERENCIA nas portas Fa0/15-16. Eleição manual de Root Bridge: SW-TI com prioridade 4096 (primário) e SW-GERENCIA com 8192 (secundário). Migração de PVST para RSTP em todos os switches. PortFast e BPDU Guard aplicados em todas as portas de acesso. Validação de failover: com SW-TI isolado, SW-GERENCIA assumiu o Root em menos de 2 segundos.

**Comandos principais:** `channel-group 1 mode active`, `spanning-tree vlan x priority`, `spanning-tree mode rapid-pvst`, `spanning-tree portfast`, `spanning-tree bpduguard enable`, `show etherchannel summary`, `show spanning-tree summary`

---

### Lab 6 — ACLs e segurança
Política least privilege entre departamentos: cada VLAN acessa apenas a própria rede e a internet — sem acesso cruzado entre filiais. ACLs estendidas aplicadas nas sub-interfaces do R1 no sentido inbound. Inclusão de regra para tráfego DHCP (UDP porta 67) no topo de cada ACL para não bloquear renovação de IPs.

**Comandos principais:** `ip access-list extended NOME`, `permit udp any host 255.255.255.255 eq 67`, `deny ip`, `permit ip`, `ip access-group NOME in`, `show access-lists`

---

### Lab 7 — NAT (PAT)
PAT configurado no R1 traduzindo todas as redes internas para o IP público 200.1.1.1. ACL padrão definindo quais redes são traduzidas. Interfaces marcadas como inside/outside. Validação em tempo real com múltiplos PCs de VLANs diferentes acessando o ISP simultaneamente.

**Comandos principais:** `ip access-list standard NAT-PERMITIDOS`, `ip nat inside source list overload`, `ip nat inside`, `ip nat outside`, `show ip nat translations`, `show ip nat statistics`

---

### Lab 8 — Redes sem fio (WLC e APs)
APs configurados com SSIDs e senhas WPA2-PSK exclusivos por prédio, canais não sobrepostos (1, 6, 11) para evitar interferência. WLC-PT adicionado à topologia conectado ao SW-TI na VLAN 10. Conceitos de CAPWAP, roaming transparente e diferença entre AP autônomo e lightweight documentados.

**Configuração dos APs:**

| AP | SSID | Canal | Senha |
|---|---|---|---|
| AP-TI | WIFI-TI | 1 | senhaIT@123 |
| AP-GERENCIA | WIFI-GERENCIA | 6 | senhaGER@123 |
| AP-RH | WIFI-RH | 11 | senhaRH@123 |
| AP-ENGENHARIA | WIFI-ENGENHARIA | 1 | senhaENG@123 |

---

### Lab 9 — RIP e EIGRP
RIP v2 configurado no R1 e R2 com `no auto-summary`. EIGRP AS 100 configurado no R2 com K values padrão (banda + delay). Comparativo completo entre RIP, EIGRP e OSPF — métricas, distância administrativa, convergência e limitações. Estudo da distância administrativa como critério de desempate quando dois protocolos aprendem a mesma rota.

**Comandos principais:** `router rip`, `version 2`, `no auto-summary`, `router eigrp 100`, `show ip protocols`, `show ip route rip`

---

### Lab 10 — Revisão final e troubleshooting
Checklist geral da topologia, verificação de todas as VLANs, DHCP, ACLs e NAT. Resolução de conflitos DHCP por impressoras com IP estático. Troubleshooting do IP Phone. Simulação de 5 questões de prova CCNA com 100% de acerto.

**Comandos principais:** `show ip dhcp conflict`, `clear ip dhcp conflict *`, `show ip dhcp binding`, `show ip nat translations`, `show access-lists`, `show spanning-tree summary`

---

## Troubleshooting real documentado

**Lab 4 — IP Phone travado em "Configuring CM List"**
O telefone tinha IP, VLAN certa, trunk funcionando e DHCP opção 150 configurada. Tudo parecia correto. O problema era um único comando ausente: `button 1:1` no ephone. Sem ele o telefone registrava no Call Manager mas não sabia qual ramal atender e ficava em loop.

**Lab 4 — Sub-interface fantasma Gig0/1.10**
Uma sub-interface sem IP na Gig0/1 criava conflito de roteamento com a Gig0/0.10 para a VLAN 10. PCs da TI não conseguiam pegar IP via DHCP. Corrigido com `no interface GigabitEthernet0/1.10`.

**Lab 5 — EtherChannel Po1 caindo por native VLAN mismatch**
O Po1 ficava em estado SD (Suspended/Down) com as portas em modo I (stand-alone). Causa: native VLAN diferente nos dois lados (VLAN 1 no SW-TI, VLAN 20 no SW-GERENCIA). Solução: destruir e recriar o canal com `no channel-group 1`, `no interface port-channel 1` e recriar com `switchport trunk native vlan 1` nos dois lados.

**Lab 6 — ACL bloqueando DHCP**
Após aplicar as ACLs, PCs de Gerência, RH e Engenharia pararam de receber IP via DHCP. Causa: quando um PC ainda não tem IP, o pedido DHCP sai com origem `0.0.0.0` — nenhuma regra da ACL permitia esse tráfego e ele caía no `deny any` implícito. Solução: adicionar `permit udp any host 255.255.255.255 eq 67` no topo de cada ACL.

**Lab 9 — Conflito de IP entre R1 e R2 via SW-TI**
R1 (`10.0.0.5/30`) e R2 (`10.0.0.2/30`) estavam em redes `/30` diferentes — nunca se comunicariam diretamente. Corrigido alterando o R2 para `10.0.0.6/30`, colocando os dois na mesma rede `10.0.0.4/30`.

**Lab 10 — Conflito DHCP por impressoras com IP estático**
Mensagens `PING_CONFLICT` aparecendo no R1 para `192.168.30.10` e `192.168.40.10`. As impressoras de RH e Engenharia estavam com IP estático não excluído do pool DHCP. Solução: `ip dhcp excluded-address` para os IPs fixos e `clear ip dhcp conflict *` para limpar o histórico.

---

## Comandos por categoria

### VLANs e switching
```
vlan 10 / name TI                        — cria e nomeia VLAN
switchport mode access                    — porta de acesso
switchport access vlan 10                 — atribui porta à VLAN
switchport voice vlan 10                  — VLAN de voz (IP Phone)
switchport mode trunk                     — porta trunk
switchport trunk native vlan 1            — define native VLAN
show vlan brief                           — lista VLANs e portas
show interfaces trunk                     — detalhes das trunks
```

### EtherChannel e STP
```
channel-group 1 mode active               — LACP modo active
no interface port-channel 1               — remove Po1
spanning-tree vlan x priority 4096        — define prioridade Root
spanning-tree mode rapid-pvst             — ativa RSTP
spanning-tree portfast                    — PortFast na porta
spanning-tree bpduguard enable            — BPDU Guard na porta
show etherchannel summary                 — estado do EtherChannel
show spanning-tree summary                — resumo do STP
```

### DHCP e VoIP
```
ip dhcp excluded-address x.x.x.x         — exclui IP do pool
ip dhcp pool NOME                         — cria pool
network / default-router / dns-server     — parâmetros do pool
option 150 ip x.x.x.x                    — endereço do Call Manager
telephony-service                         — habilita VoIP
ephone-dn 1 / number 1001                 — cria ramal
ephone 1 / mac-address / button 1:1       — associa telefone
show ip dhcp binding                      — IPs concedidos
show ip dhcp conflict                     — conflitos DHCP
```

### ACL e NAT
```
ip access-list extended NOME              — ACL estendida
permit udp any host 255.255.255.255 eq 67 — permite DHCP
deny ip origem wildcard destino wildcard  — bloqueia tráfego
permit ip origem wildcard any             — permite internet
ip access-group NOME in                   — aplica ACL inbound
ip nat inside source list NOME interface overload — PAT
ip nat inside / ip nat outside            — marca interfaces
show access-lists                         — regras e matches
show ip nat translations                  — traduções ativas
```

### Roteamento
```
router ospf 1 / network x.x.x.x wild area 0  — OSPF
router rip / version 2 / no auto-summary      — RIP v2
router eigrp 100 / network x.x.x.x           — EIGRP
show ip route                                  — tabela de rotas
show ip protocols                              — protocolos ativos
```

---

## Política de segurança (ACLs)

| De \ Para | TI | Gerência | RH | Engenharia | Internet |
|---|---|---|---|---|---|
| TI | ✅ | ✅ | ✅ | ✅ | ✅ |
| Gerência | ❌ | ✅ | ❌ | ❌ | ✅ |
| RH | ❌ | ❌ | ✅ | ❌ | ✅ |
| Engenharia | ❌ | ❌ | ❌ | ✅ | ✅ |

---

## Sobre o autor

Supervisor de TIC na Minsait, atuando na Petrobras REFAP. Tecnólogo em Segurança da Informação pela Uniasselvi. Certificações: ITIL V4, HDI, NSE1/NSE2/NSE3, CCST Networking.

Preparação para CCNA 200-301 (julho/2026) e NSE4 Fortinet (dezembro/2026). Objetivo: especialização em firewall e arquitetura de redes.

---

*Estudando com o professor Gustavo Kalau no YouTube.*
