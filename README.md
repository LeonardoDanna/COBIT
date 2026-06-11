# Banca COBIT 5 — Simulador de Prova Oral

Aplicação web de página única (`index.html`) que simula uma banca de prova oral sobre **COBIT 5**. O usuário assume o papel de diretor de uma empresa fictícia, analisa um estudo de caso gerado por IA, escolhe até 3 processos COBIT 5 que melhor resolvem o problema e defende sua escolha por escrito. A "banca" (Claude) avalia a resposta, dá uma nota de 0 a 10 e o usuário ganha XP, sobe de patente e desbloqueia conquistas.

## Como executar

Este projeto foi feito para rodar como **Artifact do Claude.ai**: o arquivo `index.html` usa `window.claude.complete(prompt)` para gerar os casos e avaliar as respostas, API disponível apenas dentro do ambiente de Artifacts (não funciona se aberto como um arquivo HTML comum em um navegador, pois essa API não existe fora desse ambiente).

Para usar:

1. Abra o conteúdo de `index.html` como um Artifact HTML no Claude.ai.
2. Clique em **"Iniciar prova oral"**.
3. Leia o caso, marque até 3 processos COBIT 5 e escreva sua justificativa.
4. Envie a resposta e veja o veredito da banca.

## Funcionalidades

- Geração dinâmica e infinita de estudos de caso, com dificuldade crescente.
- Avaliação qualitativa da justificativa do usuário, processo a processo.
- Sistema de XP, patentes (de "Estagiário de TI" a "CIO") e sequência de acertos.
- Conquistas (badges) por marcos de desempenho.
- Persistência de progresso entre sessões.

## Documentação

Para detalhes sobre a arquitetura, o fluxo do jogo, as fórmulas de pontuação e os prompts usados, veja [FUNCIONAMENTO.md](FUNCIONAMENTO.md).

<img width="949" height="699" alt="image" src="https://github.com/user-attachments/assets/2ad37cec-0194-4bd6-981f-bee8f5753d5b" />
<img width="1448" height="868" alt="image" src="https://github.com/user-attachments/assets/e5538b55-bb77-45f9-aa5b-c1fc1eb7b76e" />
<img width="1447" height="865" alt="image" src="https://github.com/user-attachments/assets/07cd49ec-79cf-4b3d-9dc2-21330ed9e220" />
<img width="1449" height="868" alt="image" src="https://github.com/user-attachments/assets/4718f995-0af3-4407-a187-3e62888490d8" />
