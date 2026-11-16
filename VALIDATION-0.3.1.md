# Correção e validação — CLA Drums 0.3.1

## Defeito reproduzido

O reverb CLUB/Large da versão 0.3.0 tinha um laço externo que devolvia sua saída wet à entrada com ganho 0,22. O ganho agregado dos filtros paralelos do reverb podia tornar esse laço maior que a unidade. A energia crescia sem sinal novo, inclusive enquanto o send estava baixo ou o gate ocultava a cauda. Subir o fader ou reabrir o gate tornava a oscilação audível.

O teste de isolamento seleciona Room/CLUB, desativa EQ, compressor e gate, coloca entrada/saída em 0 dB e reverb em +10. Aplica um único impulso de amplitude 0,1 nos dois canais a 48 kHz e depois silêncio. A versão antiga falhou no quarto segundo. Medições de uma execução:

| Segundo | Pico antigo | RMS antigo | Pico corrigido | RMS corrigido |
| --- | ---: | ---: | ---: | ---: |
| 1 | 0,0500 | 0,000785646 | 0,0500 | 0,000238048 |
| 2 | 0,0281771 | 0,00569386 | 0,0000143021 | 0,00000150809 |
| 3 | 0,372334 | 0,0862574 | 0,000000670975 | 0,0000000617567 |
| 4 | 5,4838 | 1,38944 | 0,0000000723633 | 0,0000000145248 |

Os testes anteriores processavam trechos curtos e não detectavam o crescimento da realimentação.

## Correções

- Removido o laço externo de realimentação. A recirculação ocorre somente dentro das redes limitadas do reverb JUCE; o maior feedback usado é 0,9576.
- Ganho wet normalizado conforme o tamanho da sala e a quantidade de canais; filtro de DC de 20 Hz somente na saída wet. O nível da ambiência muda em relação às versões defeituosas.
- Três engines independentes, sempre avançando, com crossfade de 20 ms. A troca de cor não reinicializa abruptamente uma cauda ativa nem revela um laço parado.
- Predelay interpolado e suavizado. O delay de Cowbell mantém feedback de 30% e suaviza mudanças de tempo/subdivisão por 30 ms.
- Bypass interno e bypass do host mantêm o DSP avançando, entregando o áudio original, para impedir que ecos antigos reapareçam ao reativar o plugin.
- Entradas NaN/Inf são descartadas antes dos filtros. Dados finitos fora do headroom de entrada de ±16 são limitados. Modos e parâmetros inválidos são limitados antes de indexar tabelas; taxas inválidas usam uma configuração segura.

Os IDs do plugin e dos parâmetros, os 26 presets e o formato de estado foram mantidos. Não foi acrescentado um limiter para mascarar o defeito. Entrada, EQ e saída ainda podem causar clipping normal se o usuário exceder o headroom; os indicadores continuam sinalizando isso.

## Verificações executadas

- Regressão de 30 segundos do impulso que fazia o reverb antigo oscilar.
- 108 combinações de 6 peças × 3 reverbs × mono/estéreo × 44,1/48/96 kHz, cada uma com 24 segundos; exigência de cauda audível no início e abaixo de -100 dBFS no trecho final.
- Mais 6 combinações Room × 3 reverbs × mono/estéreo em 192 kHz, com a mesma duração e exigência de decaimento.
- Todos os 26 presets, com automação de send/cor do reverb, Direct e bypass.
- Cenário Room – Gated 80s com o send abaixado durante processamento e depois elevado ao máximo enquanto novos transientes reabrem o gate.
- Bypass interno e do host: silêncio deve permanecer inalterado e não pode ressuscitar um eco congelado.
- NaN/Inf e entradas extremas, parâmetros brutos inválidos, blocos de tamanho zero/maiores que o anunciado e taxas de amostragem inválidas/limítrofes.
- Varredura de toda a faixa de faders, peças e cores, seguida por silêncio para exigir que a cauda desapareça.
- Testes anteriores de presets, serialização de estado, mapeamento Gate/Delay, GUI, undo/redo, A/B, escala, polaridade, ganho, harmônicos, sincronismo BPM e exclusividade por modo.

A suíte de estabilidade possui 151 cenários, além da regressão inicial do impulso. Os 150 cenários anteriores à varredura final e a suíte funcional passaram sob AddressSanitizer e UndefinedBehaviorSanitizer, com detecção de leaks ativada. A varredura final também passou sob os mesmos sanitizadores em execução separada. Nenhum erro foi reportado nessas execuções. Foram instrumentados os arquivos do processador, editor e testes e os headers neles instanciados; os módulos de terceiros não foram todos instrumentados.

## Reprodução

```sh
./scripts/build-cloud.sh linux
./build-linux7/DrumsStabilityTests --repro  # imprime pico/RMS por segundo
./build-linux7/DrumsStabilityTests --sweep  # varredura de controles
```

Para diagnóstico nativo GCC/Clang: configure CMake com `-DDRUMS_SANITIZE_PROCESSOR=ON`, compile `DrumsTests` e `DrumsStabilityTests` e rode CTest com `ASAN_OPTIONS=halt_on_error=1:detect_leaks=1` e `UBSAN_OPTIONS=halt_on_error=1:print_stacktrace=1`. Desative a opção para builds de distribuição; `build-cloud.sh` faz isso explicitamente.

## Limites

Os testes de DSP, GUI e sanitizadores foram executados no Linux. O VST3 de distribuição foi compilado para Windows x64 e deve ser validado pelo usuário na DAW Windows. Os testes não demonstram ausência de toda falha possível, não validam todas as DAWs e não comprovam fidelidade sonora ao produto original. O binário de distribuição não depende dos sanitizadores.

Para atualizar: feche a DAW, substitua a pasta inteira `CLA Drums.vst3` e reescaneie os plugins. Não continue usando as versões 0.2/0.3 com o reverb defeituoso.
