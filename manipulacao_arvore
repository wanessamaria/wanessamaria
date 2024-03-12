#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "huffunctions.h"
#include "..\estruturaLinkedList\linked.h"

// Estrutura de um nó da árvore binária
struct BinaryTreeNode {
    unsigned char data;
    struct BinaryTreeNode* left;
    struct BinaryTreeNode* right;
};

// Funções relacionadas à árvore binária
struct BinaryTreeNode* createBinaryTreeNode(unsigned char data) {
    struct BinaryTreeNode* newNode = (struct BinaryTreeNode*)malloc(sizeof(struct BinaryTreeNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

void insertBinaryTreeNode(struct BinaryTreeNode** root, unsigned char data) {
    if (*root == NULL) {
        *root = createBinaryTreeNode(data);
    } else if (data < (*root)->data) {
        insertBinaryTreeNode(&((*root)->left), data);
    } else {
        insertBinaryTreeNode(&((*root)->right), data);
    }
}

void freeBinaryTree(struct BinaryTreeNode* root) {
    if (root != NULL) {
        freeBinaryTree(root->left);
        freeBinaryTree(root->right);
        free(root);
    }
}

// Função para escrever a árvore binária em um arquivo
void writeBinaryTree(FILE *file, struct BinaryTreeNode* tree) {
    if (tree != NULL) {
        fprintf(file, "%c", tree->data);
        if (tree->left != NULL || tree->right != NULL) {
            fprintf(file, "1"); // Indica que o nó tem filhos
            writeBinaryTree(file, tree->left);
            writeBinaryTree(file, tree->right);
        } else {
            fprintf(file, "0"); // Indica que o nó é uma folha
        }
    }
}

// Função para ler a árvore binária de um arquivo
struct BinaryTreeNode* readBinaryTree(FILE *file) {
    unsigned char data;
    fscanf(file, "%c", &data);
    if (data == '\0') {
        return NULL;
    }

    struct BinaryTreeNode* newNode = createBinaryTreeNode(data);

    char hasChildren;
    fscanf(file, "%c", &hasChildren);
    if (hasChildren == '1') {
        newNode->left = readBinaryTree(file);
        newNode->right = readBinaryTree(file);
    } else {
        newNode->left = NULL;
        newNode->right = NULL;
    }

    return newNode;
}

// Funções originais do Huffman
void printByteBinary(char *binary, unsigned char byte) {
    for (int i = 7; i >= 0; i--) {
        binary[7 - i] = (byte >> i) & 1 ? '1' : '0';
    }
}

void writeNewByte(DefaultNode *node, FILE *file) {
    if (node == NULL) return;
    writeNewByte(node->next, file);
    fprintf(file, "%d", node->item);
    free(node); // Libera toda a lista.
}

void searchBinaryTree(FILE *fileEncrypty, struct BinaryTreeNode* tree, int byte, char* path) {
    if (tree != NULL) {
        if (tree->data == byte) {
            fprintf(fileEncrypty, "%s", path);
            return;
        } else {
            char leftPath[strlen(path) + 2];
            char rightPath[strlen(path) + 2];
            sprintf(leftPath, "%s0", path);
            sprintf(rightPath, "%s1", path);
            searchBinaryTree(fileEncrypty, tree->left, byte, leftPath);
            searchBinaryTree(fileEncrypty, tree->right, byte, rightPath);
        }
    }
}

void searchNewBinary(FILE *fileEncrypty, struct BinaryTreeNode* tree, int byte, DefaultNode *nodeBinary) {
    char path[256]; // Tamanho máximo do caminho
    path[0] = '\0'; // Inicializa o caminho vazio
    searchBinaryTree(fileEncrypty, tree, byte, path);
}

int leFrequencia(int *array, char *minhaString) {
    memset(array, 0, 256 * sizeof(int));
    FILE *file = fopen(minhaString, "rb");
    if (file == NULL) {
        perror("Error");
        return 0;
    }

    unsigned char byte;
    while (fscanf(file, "%c", &byte) != EOF) {
        array[byte]++;
    }

    fclose(file);
    return 1;
}

void escreverNovoBin(char *minhaString, struct BinaryTreeNode* tree) {
    FILE *file = fopen(minhaString, "rb");
    FILE *fileEncrypty = fopen("encrypted.txt", "wb");
    unsigned char byte;
    while (fscanf(file, "%c", &byte) != EOF) {
        searchNewBinary(fileEncrypty, tree, byte, NULL);
    }
    fclose(file);
    fclose(fileEncrypty);
}

void preOrderTree(FILE *file, struct BinaryTreeNode* tree) {
    if (tree != NULL) {
        fprintf(file, "%c", tree->data);
        preOrderTree(file, tree->left);
        preOrderTree(file, tree->right);
    }
}

void tamanhoDaArvore(struct BinaryTreeNode* tree, int *tamanho) {
    if (tree != NULL) {
        (*tamanho)++;
        tamanhoDaArvore(tree->left, tamanho);
        tamanhoDaArvore(tree->right, tamanho);
    }
}

int main() {
    int frequencia[256];
    printf("Digite o arquivo a ser compactado: ");
    char minhaString[30];
    scanf("%s", minhaString);

    if (!leFrequencia(frequencia, minhaString)) return 0;
    printf("Lendo a frequência de cada byte...\n");

    Priority_Queue *pq = create_priority_queue();
    printf("Criando a fila de prioridade das frequências...\n");
    criarFila(frequencia, pq);
    printf("Criando a árvore de Huffman...\n");
    criarArvoreDeHuffman(pq);
    printf("Escrevendo o novo binário do arquivo, agora compactado, resultado em: encrypted.txt\n");
    escreverNovoBin(minhaString, pq->head);

    printf("Escrevendo o cabeçalho do arquivo...\n");
    FILE *file = fopen("encrypted.txt", "r");
    unsigned long int total_bits = 0;
    while (fgetc(file) != EOF)
        total_bits++;
    fclose(file);

    short lixo = total_bits % 8;
    lixo = 8 - lixo;
    printf("\nTamanho total dos bits: %lu, Lixo: %d\n", total_bits, lixo);

    int tamanho = 0;
    tamanhoDaArvore(pq->head, &tamanho);

    FILE *fileHeader = fopen("header.txt", "wb");
    unsigned char byte1, byte2;
    byte1 = lixo << 5 | tamanho >> 8;
    byte2 = tamanho;
    char binary1[9] = { '\0' };
    char binary2[9] = { '\0' };
    printByteBinary(binary1, byte1);
    printByteBinary(binary2, byte2);
    for (int x = 0; x < 16; x++) {
        if (x < 8) {
            fputc(binary1[x], fileHeader);
        } else {
            fputc(binary2[x - 8], fileHeader);
        }
    }
    preOrderTree(fileHeader, pq->head);
    file = fopen("encrypted.txt", "rb");
    unsigned char ch;
    while (fscanf(file, "%c", &ch) != EOF) {
        fputc(ch, fileHeader);
    }
    fclose(file);

    if (lixo > 0) {
        for (int i = 0; i < lixo; i++) {
            fputc('0', fileHeader);
        }
    }

    fclose(fileHeader);

    // Escrever a árvore binária em um arquivo
    FILE *treeFile = fopen("binary_tree.txt", "w");
    writeBinaryTree(treeFile, pq->head);
    fclose(treeFile);

    // Ler a árvore binária de um arquivo
    FILE *readTreeFile = fopen("binary_tree.txt", "r");
    struct BinaryTreeNode* newRoot = readBinaryTree(readTreeFile);
    fclose(readTreeFile);

    // Liberação de memória
    freeAllTree(pq->head);
    free(pq);

    return 0;
}
