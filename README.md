# Workshop Red Hat Academy com Ansible

Repositório prático para o workshop do **Red Hat Academy / Cesar School**.

---

## 📌 Conteúdo da Aula: Role `webserver` com Template Jinja2

Esta role automatiza a instalação e configuração de um servidor web (Apache `httpd` em sistemas da família Red Hat / `apache2` em sistemas Debian/Ubuntu) e substitui a página inicial padrão por uma página customizada e dinâmica gerada através de **Jinja2**.

### 🎯 Informações Coletadas Dinamicamente via Ansible Facts

A página inicial exibe no corpo os seguintes dados da máquina coletados durante a fase de `gather_facts`:

| Fato / Métrica | Variável Ansible (Facts) | Descrição |
|---|---|---|
| **Endereço IP** | `ansible_default_ipv4.address` / `ansible_all_ipv4_addresses` | IP padrão da interface de rede ativa |
| **Versão do SO** | `ansible_distribution` e `ansible_distribution_version` | Distribuição Linux (ex: Red Hat Enterprise Linux 9, Fedora, Rocky) |
| **CPU** | `ansible_processor_vcpus` / `ansible_processor_count` | Quantidade de vCPUs e modelo do processador |
| **Memória RAM** | `ansible_memtotal_mb` | Capacidade total de memória em MB e GB |
| **Hostname** | `ansible_fqdn` / `ansible_hostname` | Nome completo da máquina na rede |
| **Timestamp** | `ansible_date_time` | Data e hora exatas do provisionamento |

---

## Topologia do laboratório

O inventário usa três máquinas distintas na rede `192.168.0.0/24`:

- `lb01`: `192.168.0.10`, grupo `haproxy`, HAProxy na porta 80.
- `web01`: `192.168.0.11`, grupo `webservers`, Apache na porta 80.
- `web02`: `192.168.0.12`, grupo `webservers`, Apache na porta 80.

Os IPs são exemplos. Ajuste `ansible_host` no arquivo `inventory` e o usuário SSH
(`ansible_user=student`) conforme seu ambiente. Configure acesso SSH por chave e
permissão de sudo nos três servidores. Use `--ask-become-pass` se o sudo exigir senha.

O `playbook.yml` aplica a role `webserver` somente ao grupo `webservers` e, em
seguida, importa `playbook_haproxy.yml`, que aplica a role `haproxy` somente ao
grupo `haproxy`. Os backends são montados automaticamente a partir do grupo
`haproxy_backend_group` (padrão: `webservers`), usando o `ansible_host`
de cada membro. No AAP, se o grupo dos webservers tiver outro nome
(ex.: `vinicius-g`), defina `haproxy_backend_group` nas variáveis do
grupo do load balancer ou no Job Template.

## Estrutura

```text
awsacademy_workshop_ansible/
├── ansible.cfg
├── inventory
├── requirements.yml
├── playbook.yml                 # Provisiona os dois webservers e o HAProxy
├── playbook_haproxy.yml         # Provisiona somente o grupo haproxy
├── group_vars/
│   └── haproxy.yml              # Porta 80 e backends do laboratório
└── roles/
    ├── webserver/
    └── haproxy/
```

## Como executar

Execute os comandos a partir deste diretório:

```bash
ansible-galaxy collection install -r requirements.yml
ansible-inventory --graph
ansible-playbook playbook.yml --syntax-check
ansible-playbook playbook.yml
```

Para executar somente uma parte:

```bash
ansible-playbook playbook.yml --limit webservers
ansible-playbook playbook_haproxy.yml
```

Acesse `http://192.168.0.10` para testar o balanceamento. A página identifica o
servidor web que respondeu. Os backends também podem ser verificados diretamente:

```bash
curl http://192.168.0.10
curl http://192.168.0.11
curl http://192.168.0.12
```

## Role `haproxy`

A role instala o HAProxy, gera a configuração com Jinja2 e valida o arquivo com
`haproxy -c` antes de substituí-lo. O serviço é habilitado no boot e alterações
na configuração notificam seu handler de reinício. Cada backend possui uma
verificação de disponibilidade TCP (`check`).

As configurações deste laboratório estão em `group_vars/haproxy.yml`, que
sobrescreve os defaults da role para usar um balanceador dedicado na porta 80
e o grupo `webservers` como origem dos backends. Timeouts, algoritmo e
opção de configurar o firewall estão disponíveis em
`roles/haproxy/defaults/main.yml`.

Em sistemas Red Hat com firewalld ativo, as roles liberam suas portas HTTP.
No AWS Academy, configure os Security Groups para permitir SSH do controlador
Ansible para os três hosts, HTTP dos clientes para `lb01` e HTTP de `lb01` para
os dois servidores web. Os playbooks não alteram Security Groups da AWS.
Em hosts com SELinux enforcing, a política local também precisa permitir as
conexões do HAProxy aos backends.
