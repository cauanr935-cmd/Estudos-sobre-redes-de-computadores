# 🌐 VLANs e Inter-VLAN Routing na Prática — Guia + Laboratório (Cisco Packet Tracer)

![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen)
![Ferramenta](https://img.shields.io/badge/ferramenta-Cisco%20Packet%20Tracer-blue)
![Nível](https://img.shields.io/badge/n%C3%ADvel-iniciante%2Fintermedi%C3%A1rio-yellow)
![Licença](https://img.shields.io/badge/licença-MIT-lightgrey)

> Um laboratório completo e comentado sobre **VLANs**, **portas trunk** e **Router-on-a-Stick**, pensado para quem está começando a estudar redes de computadores (CCNA, cursos técnicos, faculdade) e quer aprender **fazendo**, não só lendo teoria.

---

## 👋 Para quem é este repositório?

- Estudantes de **redes de computadores**, **CCNA**, **cursos técnicos em informática**
- Pessoas migrando de carreira para a área de **infraestrutura/redes**
- Professores buscando um **exemplo pronto** para usar em aula
- Qualquer pessoa curiosa sobre como escolas, empresas e provedores separam o tráfego de diferentes setores em uma mesma rede física

Você **não precisa saber nada de redes previamente** para acompanhar — os conceitos são explicados abaixo antes dos comandos.

---

## 🎯 O que você vai aprender aqui

- O que é uma **VLAN** e por que ela existe
- A diferença entre porta **access** e porta **trunk**
- O que é **encapsulamento 802.1Q (dot1Q)**
- Como um único roteador pode rotear tráfego entre várias VLANs — a técnica **Router-on-a-Stick**
- Como **verificar e testar** cada etapa da configuração, em vez de só "confiar que funcionou"
- Erros comuns de digitação e sintaxe no IOS da Cisco (e como identificá-los)

---

## 📚 Conceitos explicados

### O que é uma VLAN?

Imagine um prédio com um único switch físico conectando todos os computadores. Sem VLANs, **todo mundo está no mesmo domínio de broadcast** — ou seja, qualquer mensagem de broadcast de um PC é ouvida por todos os outros, e (dependendo da configuração) todos podem, em teoria, se enxergar na rede.

Uma **VLAN (Virtual Local Area Network)** cria divisões lógicas dentro do mesmo switch físico, como se você tivesse _vários switches virtuais_ dentro de um só. Isso é usado para:

- **Segurança**: separar o tráfego do setor financeiro do tráfego da recepção, por exemplo
- **Organização**: agrupar dispositivos por função, não por localização física
- **Performance**: reduzir o tamanho dos domínios de broadcast

### Porta Access vs. Porta Trunk

| Tipo de porta | Uso típico                                        | O que carrega                                                                                                 |
| ------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Access**    | Conectada a um dispositivo final (PC, impressora) | Tráfego de **uma única VLAN**                                                                                 |
| **Trunk**     | Conectada a outro switch ou a um roteador         | Tráfego de **várias VLANs**, cada quadro marcado com uma "etiqueta" (tag) identificando de qual VLAN ele veio |

Essa "etiqueta" é o que chamamos de **encapsulamento 802.1Q (dot1Q)** — um padrão que insere um identificador de VLAN dentro do próprio quadro Ethernet.

### Por que VLANs diferentes não se enxergam sozinhas?

Cada VLAN é, na prática, uma **rede IP separada** (por exemplo, VLAN 10 = 192.168.1.0/24 e VLAN 11 = 192.168.2.0/24). Sem um dispositivo de **Camada 3** (que entenda IP e faça roteamento), essas redes simplesmente não conseguem conversar — assim como sua rede doméstica não conversa com a internet sem um roteador no meio.

### O que é Router-on-a-Stick?

É uma técnica onde **um único roteador**, usando **uma única interface física**, consegue rotear tráfego entre várias VLANs. Isso é feito criando **subinterfaces lógicas** — pequenas "fatias" virtuais da interface física, uma para cada VLAN — cada uma configurada com:

- Um identificador de VLAN (`encapsulation dot1Q <id>`)
- Um endereço IP que funciona como **gateway** daquela VLAN

O nome vem justamente disso: o roteador fica "pendurado" (_on a stick_) no switch por um único cabo, mas consegue servir de gateway para múltiplas redes ao mesmo tempo.

---

## 🖧 Topologia do laboratório

```
                         [ Roteador ]
                        Gi0/0/0
                             |
                             | (trunk — carrega VLAN 10 e VLAN 11)
                             |
                        [ Switch ]
                   Fa0/1-3        Fa0/4-6
                (access VLAN 10) (access VLAN 11)
                  /    |    \        \    |    \
            [PC9] [PC10] [PC11]  [PC12] [PC13] [PC14]
             VLAN 10 - Professores    VLAN 11 - Alunos
             192.168.1.0/24            192.168.2.0/24
```

---

## 🛠️ Configuração passo a passo

### Etapa 1 — Criar as VLANs no switch

```
Switch1(config)#vlan 10
Switch1(config-vlan)#name professores
Switch1(config-vlan)#exit
Switch1(config)#vlan 11
Switch1(config-vlan)#name alunos
Switch1(config-vlan)#exit
```

### Etapa 2 — Atribuir portas de acesso às VLANs

```
Switch1(config)#interface range fa0/1 - 3
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 10
Switch1(config-if-range)#exit

Switch1(config)#interface range fa0/4 - 6
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 11
Switch1(config-if-range)#exit
```

### Etapa 3 — Configurar a porta trunk

```
Switch1(config)#interface fastEthernet 0/7
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#switchport trunk allowed vlan 10,11
Switch1(config-if)#no shutdown
```

### Etapa 4 — Criar as subinterfaces no roteador

```
Router(config)#interface gigabitEthernet 0/0/0
Router(config-if)#no shutdown
Router(config-if)#exit

Router(config)#interface gigabitEthernet 0/0/0.10
Router(config-subif)#encapsulation dot1Q 10
Router(config-subif)#ip address 192.168.1.1 255.255.255.0
Router(config-subif)#exit

Router(config)#interface gigabitEthernet 0/0/0.11
Router(config-subif)#encapsulation dot1Q 11
Router(config-subif)#ip address 192.168.2.1 255.255.255.0
```

### Etapa 5 — Configurar o gateway nos PCs

No Packet Tracer: PC → **Desktop** → **IP Configuration** → _Default Gateway_ = IP da subinterface correspondente.

> 💾 Sempre finalize com `write memory` (ou `copy running-config startup-config`) para não perder a configuração ao reiniciar o dispositivo.

---

## ✅ Como validar cada etapa

| Comando                   | Onde     | O que verificar                                               |
| ------------------------- | -------- | ------------------------------------------------------------- |
| `show vlan brief`         | Switch   | Portas atribuídas corretamente às VLANs 10 e 11               |
| `show interfaces trunk`   | Switch   | Porta trunk ativa (`trunking`), VLANs 10 e 11 permitidas      |
| `show ip interface brief` | Roteador | Subinterfaces `.10` e `.11` com status `up/up` e IPs corretos |
| `show ip route`           | Roteador | As duas redes aparecendo como diretamente conectadas (`C`)    |

---

## 🧪 Testes de conectividade

| Teste                                   | Resultado esperado | Por quê                                             |
| --------------------------------------- | ------------------ | --------------------------------------------------- |
| PC9 → PC10 (mesma VLAN)                 | ✅ Sucesso         | Mesma rede, mesmo domínio de broadcast              |
| PC9 → 192.168.1.1 (gateway)             | ✅ Sucesso         | Confirma que o PC enxerga o roteador                |
| PC9 → PC12 **sem** roteador configurado | ❌ Falha           | VLANs são isoladas por padrão                       |
| PC9 → PC12 **com** Router-on-a-Stick    | ✅ Sucesso         | Tráfego passa pelo roteador e é roteado entre redes |

> 🔍 **Dica de diagnóstico:** repare no **TTL** da resposta do ping. Dentro da mesma VLAN, o TTL chega em 255 (nenhum roteador no caminho). Ao atravessar o roteador, o TTL cai para 127 — prova de que houve um salto de roteamento.

---

## ⚠️ Erros comuns (e como resolvê-los)

Esses foram erros reais cometidos durante a montagem deste laboratório — deixados aqui de propósito, porque **errar faz parte do aprendizado** e provavelmente você vai encontrar os mesmos:

| Erro                                                                      | Causa                                                                          | Solução                                                               |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| `% Invalid input detected` ao rodar `show running-config`                 | Comando `show` digitado dentro do modo de configuração (`config#`)             | Digite `exit` para sair do modo config antes de rodar comandos `show` |
| `% Invalid input detected` ao rodar `write memory`                        | Mesmo problema — `write memory` só funciona no modo privilegiado (`#`)         | Saia do modo config com `exit` antes                                  |
| VLAN criada mas nenhuma porta associada                                   | Esquecer de rodar `switchport access vlan <id>` em cada interface              | Sempre confirme com `show vlan brief` depois de configurar            |
| Roteador doméstico (ex: Linksys WRT300N) não aceita `encapsulation dot1Q` | Roteadores domésticos não têm CLI completa nem suportam subinterfaces          | Use um roteador Cisco de linha ISR/2900/1900 no Packet Tracer         |
| `ip adress` não reconhecido                                               | Erro de digitação (faltou uma letra)                                           | Sempre confira a sintaxe: `ip address <IP> <máscara>`                 |
| Primeiro ping "falha" para outra VLAN, os seguintes funcionam             | Comportamento normal do ARP (o dispositivo ainda não conhece o MAC de destino) | Não é erro — rode o ping novamente, ou ignore a primeira perda        |

---

## 📂 Estrutura deste repositório

```
📁 vlan-router-on-a-stick/
├── README.md                    → este guia
├── topologia.pkt                → arquivo do Packet Tracer, pronto para abrir e explorar
├── /prints
│   ├── show-vlan-brief.png
│   ├── show-interfaces-trunk.png
│   ├── show-ip-interface-brief.png
│   ├── ping-intra-vlan.png
│   └── ping-inter-vlan.png
└── /diagramas
    └── topologia.png
```

---

## 🚀 Quer ir além? Ideias para expandir este laboratório

- [ ] Aplicar **ACLs** para permitir só um sentido de comunicação entre VLANs (ex: professores acessam alunos, mas não o contrário)
- [ ] Configurar **DHCP** por VLAN no roteador, em vez de IP fixo
- [ ] Adicionar uma terceira VLAN (ex: Administração/TI)
- [ ] Usar uma **VLAN nativa** diferente da VLAN 1 (boa prática de segurança)
- [ ] Ativar **Port Security** para limitar quantos endereços MAC cada porta aceita
- [ ] Substituir o Router-on-a-Stick por um **switch de Camada 3 (multilayer)** e comparar as duas abordagens

---

## 🤝 Contribuindo

Encontrou um erro, tem uma sugestão de melhoria, ou quer adicionar uma variação deste laboratório (ex: com ACLs ou DHCP)? Fique à vontade para abrir uma _issue_ ou enviar um _pull request_. O objetivo deste repositório é ser um recurso vivo e colaborativo para quem está aprendendo redes.

## 📖 Referências para aprofundar

- Documentação oficial da Cisco sobre VLANs e trunking (search: "Cisco VLAN Trunking Protocol configuration guide")
- Material de estudo para certificação **CCNA 200-301** (tópico: VLANs e Inter-VLAN Routing)
- Cisco Networking Academy (NetAcad) — cursos gratuitos que usam o Packet Tracer

## 📄 Licença

Este projeto está sob a licença MIT — sinta-se livre para usar, adaptar e compartilhar, inclusive em contexto educacional.

---

<p align="center">Feito como parte de um estudo prático de redes de computadores. Se este material te ajudou, considere deixar uma ⭐ no repositório!</p>
