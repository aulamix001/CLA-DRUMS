# CLA Drums 0.3.1 — correção do reverb / Windows x64 VST3

**[Baixar VST3 corrigido para Windows 64 bits](https://github.com/aulamix001/CLA-DRUMS/raw/refs/heads/downloads-v0.3.1/downloads/CLA-Drums-0.3.1-Windows-x64.zip)**

As versões 0.2/0.3 tinham um laço extra de realimentação no modo CLUB que podia oscilar sozinho, inclusive com o send abaixado. O defeito foi reproduzido e corrigido. Use a versão 0.3.1.

- Laço externo removido; ganho wet normalizado e DC filtrado.
- Três engines com transição suavizada, sem expor laços inativos.
- Bypass do plugin e do host não congelam mais os ecos.
- Tratamento de NaN/Inf, parâmetros inválidos e condições de bloco/taxa.
- 26 presets, 151 cenários de estabilidade, testes funcionais e verificações ASan/UBSan passaram no Linux. Ver [relatório completo](VALIDATION-0.3.1.md).

Feche a DAW, extraia o ZIP e substitua a pasta inteira `CLA Drums.vst3` em `C:\Program Files\Common Files\VST3`. Abra a DAW e reescaneie os plugins. O ID do plugin e os IDs dos parâmetros foram mantidos. O nível wet foi recalibrado e pode soar mais baixo que nas versões defeituosas.

O arquivo é um VST3 Windows PE x86-64 real, compilado em C++17/JUCE 7.0.12. Ainda falta validar carregamento e reprodução em uma DAW Windows; os testes não demonstram ausência de toda falha possível. O binário de distribuição não depende dos sanitizadores.

A interface é a mesma da versão 0.3:

![Interface do editor](preview/CLA-Drums-0.3.0.png)

Checksums em `downloads/SHA256SUMS.txt`.
