# Implantes

Página web para registar implantes cirúrgicos numa Google Sheet. Lê o código UDI
(GS1 DataMatrix) da embalagem com a câmara do telemóvel, separa-o em GTIN, lote e
validade, e guarda uma linha por implante, associada ao NSC e à data da cirurgia.

Corre inteiramente no navegador e está alojada no GitHub Pages. Não tem servidor
próprio: os dados seguem diretamente para um Google Apps Script que escreve na folha.

## Como se usa

1. Preencher o NSC e a data da cirurgia.
2. Ler o código de cada implante de uma destas formas:
   - câmara ao vivo, apontando ao código quadrado da embalagem;
   - «Fotografar», que resulta melhor em códigos muito pequenos;
   - leitor de códigos USB ou escrita manual no campo «Código».
3. Confirmar os campos lidos e carregar em «Guardar na folha».
4. No fim, «Nova cirurgia» limpa o NSC e a lista.

Se a rede falhar, o implante fica na lista com o botão «Reenviar».

## O que é lido do código

O código é uma sequência de campos GS1, cada um precedido por um identificador (AI):

| AI   | Campo            | Formato                              |
|------|------------------|--------------------------------------|
| `01` | GTIN             | 14 dígitos, o último é de controlo   |
| `10` | Lote             | até 20 caracteres                    |
| `17` | Validade         | AAMMDD (dia `00` = último do mês)    |
| `11` | Data de fabrico  | AAMMDD                               |
| `21` | Número de série  | até 20 caracteres                    |

Outros identificadores ficam guardados na coluna «Outros». A página avisa quando o
dígito de controlo do GTIN falha, quando a validade já passou e quando o código não
é GS1 (por exemplo HIBC, que fica guardado em bruto).

## Colunas da folha

Registado em, NSC, Data da cirurgia, GTIN, Lote, Validade, Fabrico, Nº de série,
Outros, Código lido, ID.

## Instalação

### 1. Google Sheet

1. Criar uma Google Sheet.
2. Extensões > Apps Script, e colar o script `Code.gs` (não está neste repositório;
   vive apenas no Apps Script).
3. Trocar o valor de `CHAVE` por uma chave própria, comprida.
4. Implementar > Nova implementação > «Aplicação Web», a executar como «Eu» e com
   acesso «Qualquer pessoa».
5. Copiar o endereço que acaba em `/exec`.

Depois de alterar o script é preciso publicar uma versão nova em
Implementar > Gerir implementações.

### 2. GitHub Pages

1. Settings > Pages > «Deploy from a branch», ramo `main`, pasta `/ (root)`.
2. Abrir `https://<utilizador>.github.io/<repositório>/` no telemóvel.

### 3. Ligar a página à folha

Na página, tocar na roda dentada, colar o endereço `/exec` e a chave, e carregar em
«Testar ligação».

## Ficheiros

```
index.html           a aplicação inteira (HTML, CSS e JavaScript)
zxing-reader.js      biblioteca de leitura de códigos
zxing_reader.wasm    idem, parte em WebAssembly
```

## Privacidade e segurança

- O endereço do Apps Script e a chave ficam guardados só no navegador de cada
  telemóvel. **Não os escrevas em nenhum ficheiro deste repositório**, que é público.
- O NSC e os implantes da cirurgia em curso ficam guardados no navegador até
  carregares em «Nova cirurgia».
- O NSC associado a um implante é um dado de saúde identificável. A folha de destino
  deve estar numa conta autorizada pela instituição para esse fim.
- A chave impede que terceiros escrevam na folha, mas quem a tiver pode fazê-lo.
  Para a mudar, altera `CHAVE` no script e publica uma versão nova.

## Limitações

- A câmara só funciona em HTTPS (o GitHub Pages cumpre).
- Códigos HIBC não são separados em campos.
- O ano das datas é assumido como 20AA.

## Créditos

Leitura de códigos com [zxing-wasm](https://github.com/Sec-ant/zxing-wasm) 3.1.6
(licença MIT), baseada em zxing-cpp.
