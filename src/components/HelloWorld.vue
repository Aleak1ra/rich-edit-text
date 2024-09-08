<template>
  <v-container>
    <!-- Menu de Edição -->
    <v-row class="editor-menu d-flex">
      <!-- Botão para Negrito -->
      <v-btn @click="toggleBold" icon>
        <v-icon>mdi-format-bold</v-icon>
      </v-btn>

      <!-- Botão para selecionar a fonte -->
      <v-select
        v-model="selectedFont"
        :items="fontsList"
        label="Escolha a fonte"
        @change="applyFontFamily"
      >
        <template v-slot:prepend>
          <v-icon>mdi-format-font</v-icon>
        </template>
      </v-select>

      <!-- Botões de Alinhamento -->
      <v-btn @click="setAlignment('left')" icon>
        <v-icon>mdi-format-align-left</v-icon>
      </v-btn>
      <v-btn @click="setAlignment('center')" icon>
        <v-icon>mdi-format-align-center</v-icon>
      </v-btn>
      <v-btn @click="setAlignment('right')" icon>
        <v-icon>mdi-format-align-right</v-icon>
      </v-btn>
      <v-btn @click="setAlignment('justify')" icon>
        <v-icon>mdi-format-align-justify</v-icon>
      </v-btn>

      <!-- Botão para adicionar Imagem -->
      <v-btn icon>
        <v-file-input
          hide-input
          label="Adicionar Imagem"
          accept="image/*"
          @change="insertImage"
        >
          <template v-slot:prepend>
            <v-icon>mdi-image</v-icon>
          </template>
        </v-file-input>
      </v-btn>

      <v-btn @click="insertDiv" icon>
        <v-icon>mdi-square-outline</v-icon>
      </v-btn>

      <!-- Botão para adicionar a tabela -->
      <v-btn @click="insertTable" icon>
        <v-icon>mdi-table-large</v-icon>
      </v-btn>

      <!-- Seletor de Cores de Fundo -->
      <v-select
        v-model="selectedBgColor"
        :items="colorOptions"
        label="Cor de Fundo"
        @change="applyCellBackgroundColor"
      >
        <template v-slot:prepend>
          <v-icon>mdi-format-color-fill</v-icon>
        </template>
      </v-select>

      <v-btn
        :class="{ 'bg-red-toggle': isRedBackground }"
        @click="toggleRedBackground"
        icon
      >
        <v-icon>mdi-format-color-fill</v-icon>
      </v-btn>
    </v-row>

    <!-- Painel de Texto -->
    <v-row>
      <div
        id="textEditor"
        class="text-panel"
        contenteditable="true"
        ref="textEditor"
        @input="onTextInput"
        @keydown="handleKeyDown"
      ></div>
    </v-row>
  </v-container>
</template>

<style scoped>
.editor-menu {
  margin-bottom: 10px;
}

.text-panel {
  border: 1px solid #ccc;
  padding: 10px;
  /* font-family: Arial, Helvetica, sans-serif; */
  min-height: 200px;
  width: 100%;
  background-color: #fff;
  font-size: 16px;
  outline: none;
  color: #000;
  white-space: pre-wrap; /* Preserve espaços e quebras de linha */
}

.bg-red-toggle {
  background-color: red !important;
  color: white; /* Ajusta a cor do ícone para garantir visibilidade */
}
</style>

<script setup>
import { ref, watch } from "vue";

const textEditor = ref(null);
const selectedFont = ref("Courier New");
const selectedBgColor = ref("");
const savedSelection = ref(null);
const fontsList = [
  "Arial",
  "Courier New",
  "Georgia",
  "Times New Roman",
  "Goudy Bookletter 1911",
  "Gill Sans Extrabold",
  "Verdana",
];
const colorOptions = ["Yellow", "LightBlue", "LightGreen"];

// Variáveis para armazenar o estado do redimensionamento
let currentTD = null;
let startX = 0;
let startWidth = 0;
let startY = 0;
let startHeight = 0;

// Observe mudanças na fonte selecionada e aplique-a ao painel de texto
// watch(selectedFont, (newFont) => {
//   if (textEditor.value) {
//     textEditor.value.style.fontFamily = newFont;
//   }
// });

// Estado para modo de negrito
let isBoldMode = ref(false);

// Função para alternar entre negrito e texto normal
const toggleBold = () => {
  const selection = window.getSelection();
  if (selection.rangeCount > 0 && !selection.isCollapsed) {
    // Se há uma seleção de texto, aplicamos negrito apenas nela
    document.execCommand("bold");
  } else {
    // Se não há seleção, alteramos o modo para o próximo texto digitado
    isBoldMode.value = !isBoldMode.value;
  }
};

// Função para capturar a entrada de texto e aplicar o efeito negrito se estiver no modo bold
const onTextInput = () => {
  // Não fazemos nada se o modo negrito não estiver ativo
  if (!isBoldMode.value) return;

  // Recupera a seleção e o intervalo de inserção
  const selection = window.getSelection();
  const range = selection.getRangeAt(0);

  // Cria um span com estilo negrito
  const boldSpan = document.createElement("span");
  boldSpan.style.fontWeight = "bold";

  // Insere o span e posiciona o cursor dentro dele
  range.surroundContents(boldSpan);

  // Posiciona o cursor após o span
  range.setStartAfter(boldSpan);
  selection.removeAllRanges();
  selection.addRange(range);
};

const handleKeyDown = (event) => {
  if (event.key === "Backspace" || event.key === "Delete") {
    return; // Não aplicamos negrito ao deletar
  }

  if (isBoldMode.value) {
    document.execCommand("bold");
  }
};

// Função para aplicar a cor de fundo ao texto selecionado
const applyCellBackgroundColor = () => {
  const color = selectedBgColor.value;
  if (!color) return;

  const selection = window.getSelection();
  if (selection.rangeCount > 0) {
    const range = selection.getRangeAt(0);

    // Verifica se há texto selecionado
    if (!range.collapsed) {
      const fragment = range.extractContents(); // Extrai o conteúdo selecionado
      const bgSpan = document.createElement("span");
      bgSpan.style.backgroundColor = color;
      bgSpan.style.display = "inline"; // Faz com que o span não quebre a linha
      bgSpan.appendChild(fragment); // Adiciona o conteúdo ao span

      range.insertNode(bgSpan); // Insere o span no local do cursor

      // Ajusta o cursor após o span
      range.setStartAfter(bgSpan);
      range.collapse(true);
      selection.removeAllRanges();
      selection.addRange(range);
    }
  }
};

// Função para aplicar o alinhamento
const setAlignment = (align) => {
  const selection = window.getSelection();
  if (!selection.rangeCount) return;

  const range = selection.getRangeAt(0);
  const block = document.createElement("div");
  block.style.textAlign = align;
  const fragment = range.extractContents();
  block.appendChild(fragment);
  range.insertNode(block);
};

const insertDiv = () => {
  const selection = window.getSelection();
  if (selection.rangeCount > 0) {
    const range = selection.getRangeAt(0);

    // Cria a div com estilo de borda e altura/largura
    const div = document.createElement("div");
    div.style.height = "100px";
    div.style.width = "100%";
    div.style.padding = "10px";
    div.style.border = "1px solid black";
    div.style.boxSizing = "border-box"; // Garante que a borda seja incluída na largura total

    // Insere a div no local do cursor
    range.deleteContents(); // Remove qualquer conteúdo selecionado, se houver
    range.insertNode(div);

    // Coloca o cursor logo após a div
    range.setStartAfter(div);
    range.collapse(true);
    selection.removeAllRanges();
    selection.addRange(range);
  }
};

const insertTable = () => {
  const selection = window.getSelection();
  if (selection.rangeCount > 0) {
    const range = selection.getRangeAt(0);

    // Cria a tabela com altura de 200px e largura total
    const table = document.createElement("table");
    table.style.width = "100%";
    table.style.height = "30px";
    table.style.borderCollapse = "collapse";
    table.style.border = "1px solid black";

    // Adiciona uma linha à tabela
    const tr = document.createElement("tr");

    // Adiciona células à linha (exemplo de 3 células)
    for (let i = 0; i < 3; i++) {
      const td = document.createElement("td");
      td.style.border = "1px solid black";
      td.style.textAlign = "center";
      td.style.position = "relative"; // Para permitir adicionar o grip de redimensionamento

      // Criar resizer horizontal
      const resizerX = document.createElement("div");
      resizerX.style.width = "5px";
      resizerX.style.height = "100%";
      resizerX.style.position = "absolute";
      resizerX.style.top = "0";
      resizerX.style.right = "0";
      resizerX.style.cursor = "col-resize";
      resizerX.style.backgroundColor = "transparent";

      resizerX.addEventListener("mousedown", initResizeX);

      // Criar resizer vertical
      const resizerY = document.createElement("div");
      resizerY.style.height = "5px";
      resizerY.style.width = "100%";
      resizerY.style.position = "absolute";
      resizerY.style.bottom = "0";
      resizerY.style.left = "0";
      resizerY.style.cursor = "row-resize";
      resizerY.style.backgroundColor = "transparent";

      resizerY.addEventListener("mousedown", initResizeY);

      td.appendChild(document.createTextNode(`Cell ${i + 1}`));
      td.appendChild(resizerX);
      td.appendChild(resizerY);
      tr.appendChild(td);
    }

    table.appendChild(tr);

    // Insere a tabela no local do cursor
    range.deleteContents();
    range.insertNode(table);

    // Coloca o cursor após a tabela
    range.setStartAfter(table);
    range.collapse(true);
    selection.removeAllRanges();
    selection.addRange(range);
  }
};
// Inicia o redimensionamento horizontal (largura) ao clicar no grip
const initResizeX = (e) => {
  currentTD = e.target.parentElement;
  startX = e.pageX;
  startWidth = currentTD.offsetWidth;

  document.addEventListener("mousemove", resizeColumnX);
  document.addEventListener("mouseup", stopResize);
};

// Inicia o redimensionamento vertical (altura) ao clicar no grip
const initResizeY = (e) => {
  currentTD = e.target.parentElement;
  startY = e.pageY;
  startHeight = currentTD.offsetHeight;

  document.addEventListener("mousemove", resizeColumnY);
  document.addEventListener("mouseup", stopResize);
};

// Função que redimensiona a largura da célula
const resizeColumnX = (e) => {
  const newWidth = startWidth + (e.pageX - startX);
  currentTD.style.width = newWidth + "px";
};

// Função que redimensiona a altura da célula
const resizeColumnY = (e) => {
  const newHeight = startHeight + (e.pageY - startY);
  currentTD.style.height = newHeight + "px";
};

// Para o redimensionamento ao soltar o mouse
const stopResize = () => {
  document.removeEventListener("mousemove", resizeColumnX);
  document.removeEventListener("mousemove", resizeColumnY);
  document.removeEventListener("mouseup", stopResize);
};

// Função para inserir imagem no local do cursor
const insertImage = (event) => {
  const selectedFile = event.target.files[0];
  if (selectedFile) {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = document.createElement("img");
      img.src = e.target.result;
      img.style.maxWidth = "100%"; // Ajusta o tamanho da imagem para a largura do painel
      img.style.width = "auto"; // Mantém a proporção original da imagem
      img.style.display = "inline"; // Exibe a imagem em linha com o texto
      img.style.margin = "10px"; // Adiciona margens ao redor da imagem
      img.style.verticalAlign = "middle"; // Alinha a imagem com o texto

      const selection = window.getSelection();
      if (selection.rangeCount > 0) {
        const range = selection.getRangeAt(0);
        range.deleteContents(); // Remove o conteúdo selecionado, se houver
        range.insertNode(img); // Insere a imagem na posição do cursor
        // Coloca o cursor logo após a imagem para permitir mais edição
        range.setStartAfter(img);
        range.collapse(true);
        selection.removeAllRanges();
        selection.addRange(range);
      } else {
        // Se não houver seleção, insere a imagem no final do editor
        textEditor.value.appendChild(img);
      }
    };
    reader.readAsDataURL(selectedFile);
  }
};

const isRedBackground = ref(false); // Estado do botão toggle

// Função para alternar a cor de fundo vermelha
const toggleRedBackground = () => {
  const color = "red"; // Cor vermelha
  const selection = window.getSelection();

  if (selection.rangeCount > 0) {
    const range = selection.getRangeAt(0);

    // Verifica se a cor de fundo vermelha já está aplicada
    const isRed = Array.from(range.cloneContents().querySelectorAll('span')).some(span => {
      return span.style.backgroundColor === color;
    });

    if (isRed) {
      // Remove a cor de fundo vermelha
      const spans = Array.from(range.cloneContents().querySelectorAll('span'));
      spans.forEach(span => {
        if (span.style.backgroundColor === color) {
          span.style.backgroundColor = ''; // Remove a cor de fundo
        }
      });
      range.deleteContents(); // Remove o conteúdo selecionado
      spans.forEach(span => {
        range.insertNode(span); // Reinsere o conteúdo modificado
      });
    } else {
      // Adiciona a cor de fundo vermelha
      const bgSpan = document.createElement("span");
      bgSpan.style.backgroundColor = color;
      bgSpan.style.display = "inline"; // Faz com que o span não quebre a linha

      const fragment = range.extractContents(); // Extrai o conteúdo selecionado
      bgSpan.appendChild(fragment); // Adiciona o conteúdo ao span com fundo vermelho
      range.insertNode(bgSpan); // Insere o span no local do cursor

      range.setStartAfter(bgSpan);
      range.collapse(true);
      selection.removeAllRanges();
      selection.addRange(range);
    }

    // Alterna o estado do botão
    isRedBackground.value = !isRedBackground.value;
  }
};
</script>
