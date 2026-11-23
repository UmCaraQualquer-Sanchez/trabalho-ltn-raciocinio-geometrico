# Logic Tensor Networks (LTNtorch): Raciocínio Geométrico

Trabalho da disciplina de **Inteligência Artificial Neuro-Simbólica**.
Extensão do tutorial de LTNtorch com relações geométricas em 2D e sólidos 3D, em que os axiomas em lógica de primeira ordem (FOL) são treinados como uma base de conhecimento fuzzy e testados em dados nunca vistos.

> **Integrantes:**
> - Antony W. P. Oliveira
> - João P. N. Seixas
> - José H. C. Costa
> - Nelson J. R. dos Santos
> - Kauã S. Viana
> - Lívia Karolina

## Arquivo principal

`MeuTrabalho_Lab_Metodologias_Ativas_19_11_26_revisado_v4.ipynb`: notebook completo, já executado, com as saídas salvas (treino e consultas).

## Como executar

**Google Colab (mais simples)**
1. Abra o notebook no Colab (Arquivo → Fazer upload de notebook).
2. Ambiente de execução → **Executar tudo**. A primeira célula instala o LTNtorch automaticamente, se necessário.

**Local (conforme o tutorial)**
```bash
conda create -n ltn_env python=3.10
conda activate ltn_env
pip install torch torchvision torchaudio
pip install "numpy==1.26.4"
pip install git+https://github.com/tommasocarraro/LTNtorch.git
jupyter notebook
```
Selecione o kernel `ltn_env`, reinicie o kernel e execute as células em ordem.

Todas as extensões usam `torch.manual_seed(42)`, então os resultados são reproduzíveis.

## O que o notebook contém

| Parte | Conteúdo |
|---|---|
| Tutorial base | Constantes, variáveis, predicados, axiomas, treino e consultas; diferentes formas de igualdade |
| **Extensão 2D** | Círculo, Quadrado, Triângulo; `LeftOf`, `Above`, `Overlaps`; axiomas da tarefa + axioma extra |
| **Sólidos 3D** | Cone, Cilindro, Paralelepípedo; relação `MaiorVolume` baseada nas fórmulas de volume |
| **Extensão 2D avançada** | Axiomas mais elaborados: transitividade, antissimetria, relações inversas, existencial |

## Axiomas (FOL)

### Extensão 2D
```
1. ∀x ∀y (Circle(x) ∧ Square(y) → LeftOf(x,y))
2. ∀x ∀y (Triangle(x) ∧ Circle(y) → Above(x,y))
3. ∀x ∀y (Overlaps(x,y) → Overlaps(y,x))
4. ∀x ¬Overlaps(x,x)
5. (extra) ∀x ∀y (Square(x) ∧ Triangle(y) → ¬Overlaps(x,y))
```

### Sólidos 3D
Fórmulas de volume: cone `V = pi*r^2*h/3`, cilindro `V = pi*r^2*h`, paralelepípedo `V = a*b*c`.
```
1. Ancoragem: Cone, Cilindro e Paralelepípedo são classes exclusivas
2. ∀x ∀y (Cilindro(x) ∧ Cone(y) ∧ mesma base e altura → MaiorVolume(x,y))
3. ∀x ∀y (MaiorVolume(x,y) → ¬MaiorVolume(y,x))                        antissimetria
4. ∀x ¬MaiorVolume(x,x)                                                 irreflexividade
5. ∀x ∀y ∀z (MaiorVolume(x,y) ∧ MaiorVolume(y,z) → MaiorVolume(x,z))    transitividade
6. ∀x (Cone(x) → ∃y (Paralelepipedo(y) ∧ MaiorVolume(y,x)))
7. Pares decididos pela fórmula de volume
```

### Extensão 2D avançada
Predicados novos: `RightOf` (inverso de `LeftOf`) e `Below` (inverso de `Above`).
```
1. Exclusão mútua das classes
2. Transitividade de LeftOf e de Above
3. Antissimetria e irreflexividade de LeftOf e de Above
4. LeftOf(x,y) ↔ RightOf(y,x)   e   Above(x,y) ↔ Below(y,x)
5. ∀c (Circle(c) → ∃s (Square(s) ∧ LeftOf(c,s)))
6. Ancoragem pelas coordenadas: x menor → LeftOf; y maior → Above
```

## Treino e generalização

Em cada extensão, os dados aleatórios de treino são gerados **uma única vez**. Depois do treino, o modelo é consultado em **novos dados aleatórios**, para mostrar que aprendeu o conceito (por exemplo, "estar à esquerda") e não decorou as coordenadas.

## Resultados (saídas do notebook)

| Parte | Satisfação final | Consultas em dados novos |
|---|---|---|
| 2D | 0.9932 | classificação 0.9996 a 1.0; `LeftOf`/`Above` corretos (1.0 vs 0.0 na ordem inversa); axiomas quantificados ≥ 0.9995 |
| Sólidos 3D | 0.9335 | classificação 0.9966 a 0.9996; cilindro > cone = 1.0 (inverso = 0.0); transitividade e antissimetria 0.9998 |
| 2D avançado | 0.9818 | classificação 0.9983 a 1.0; inversas e transitividade ≥ 0.9995; equivalência `LeftOf → RightOf` 0.9741 |

Nos testes do par triângulo/quadrado, as previsões batem com o valor verdadeiro calculado pelas coordenadas.

## Decisões de projeto

- **Ancoragem.** Axiomas só de implicação ficam verdadeiros "de graça" quando o antecedente é falso, e os classificadores colapsam para 0. Por isso as classes são ancoradas com axiomas de rotulagem (`∀c Circle(c)`, `∀c ¬Square(c)`, etc.).
- **Regiões separadas.** Os quadrados têm `y` em `[0,5]` para não invadirem a região dos triângulos (`y` em `[6,10]`). Antes, a sobreposição de regiões fazia a rede receber rótulos contraditórios.
- **Classificador de sólidos.** O tipo é decidido pela razão de preenchimento (volume / caixa envolvente): cone `pi/12`, cilindro `pi/4`, paralelepípedo `1`. Essa razão vem direto das fórmulas de volume.
- **Dados do 2D avançado.** 20 figuras por classe, para a fronteira entre regiões ficar mais firme.

## Limitações

- `Overlaps` não tem fórmula que a ensine e entra apenas como axioma de "não se sobrepõe", então tende a 0 e não é consultada diretamente.
- A satisfação dos sólidos não chega a 1.0 porque são muitos axiomas competindo (inclusive o existencial).
- Os dados são sintéticos e simples, em regiões bem separadas, pensados para ilustrar o raciocínio lógico e não para um problema real.

## Referências

- LTNtorch: <https://github.com/tommasocarraro/LTNtorch>
- Tutorial da disciplina: *Tutorial Prático: Logic Tensor Networks (LTNtorch)*, 20/11/2026.
