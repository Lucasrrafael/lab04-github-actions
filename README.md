# Laboratório 04 — GitHub Actions: controle de fluxo e deploy na AWS (CI/CD · Web Academy UFAM)

Aplicação Express ("Bem vindos ao WebAcademy!!", porta 8080) com o pipeline
[`wa-aws-ec2-docker.yml`](.github/workflows/wa-aws-ec2-docker.yml) (**AWS Deploy**), disparado a cada push:

```
lint ──► build_docker ──► Deploy AWS EC2
```

| Job | O que faz | Controle de fluxo |
|---|---|---|
| `lint` | ESLint no código | o passo `check-lint` só roda se o lint falhar (`if: failure() && steps.lint-step.outcome == 'failure'`) |
| `build_docker` | build e push da imagem [`lucasrrafael/webacademy-lab04:latest`](https://hub.docker.com/r/lucasrrafael/webacademy-lab04) | `needs: lint` |
| `deploy` | na EC2, baixa a imagem nova e recria o container | `needs: build_docker` + environment `aws_deploy` |

## Deploy na AWS

- **EC2** `t3.micro` (Amazon Linux 2023) com Docker; security group libera só a porta **8080** (sem SSH).
- O GitHub autentica na AWS por **OIDC** (`aws-actions/configure-aws-credentials`), assumindo uma role
  que só pode enviar comandos para essa instância — nenhuma chave da AWS fica guardada no GitHub.
- O deploy roda pelo **AWS Systems Manager** (Run Command): `docker pull` → `docker rm -f` → `docker run`.
- O último passo testa a aplicação publicada com `curl`.

O roteiro original usava o Google Cloud Run (passos 15 a 25, opcionais); o deploy foi feito na AWS.
