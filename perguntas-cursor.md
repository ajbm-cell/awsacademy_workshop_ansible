# Perguntas ao AAP no chat do Cursor

Os servidores MCP `aap-jobs`, `aap-inventory`, `aap-monitor`, `aap-users`, `aap-security` e `aap-platform` precisam estar verdes. Eles falam com `http://127.0.0.1:18448`, e o proxy encaminha para `https://aap.localnet.local:8448`.

Cole a pergunta no chat do Agent. O Cursor escolhe a tool. Não lance job nesta conversa a menos que a pergunta peça confirmação.

## Catálogo

```
Use o MCP aap-jobs. Liste os job templates cujo nome contém "AIOps"
e os workflow job templates com o mesmo termo.
Mostre id, nome e, se existir, o playbook.
Não lance nada.
```

```
Use o MCP aap-jobs. Quantos job templates existem no Controller
e quais não têm "AIOps" no nome?
Não lance nada.
```

## Inventário e plataforma

```
Use o MCP aap-inventory. Liste os inventários e, no inventário
haproxy_project_inventory, liste os hosts com o nome.
Não altere inventário nem crie host.
```

```
Use o MCP aap-monitor. Qual a versão e o status do Gateway?
Não altere configuração.
```

## Um job, sem executar

```
Use o MCP aap-jobs. Abra o job template "AIOps - Check Status httpd".
Diga o id, o playbook, o inventário e se ele pede limit no launch.
Não lance.
```

## Executar só depois do sim

```
Use o MCP aap-jobs. Quero lançar "AIOps - Check Status httpd".
Antes de chamar job_templates_launch_create, repita o id e o playbook
e espere eu responder "sim".
Se eu confirmar, acompanhe com jobs_retrieve e resuma o stdout
sem colar HTML.
Não dispare "AIOps - HTTPD Break (LAB ONLY)" nem o workflow.
```
