
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <locale.h>
#include <windows.h>

typedef enum {
    ENTRADA,
    PRINCIPAL,
    SOBREMESA,
    BEBIDA
} Categoria;

typedef enum {
    PENDENTE,
    EM_PREPARO,
    PRONTO,
    ENTREGUE
} StatusPedido;

typedef struct {
    char nome[100];
    char descricao[200];
    float preco;
    Categoria categoria;
} ItemMenu;

typedef struct {
    int id;
    char nomeCliente[100];
    ItemMenu *itens;
    int quantidadeItens;
    StatusPedido status;
} Pedido;

ItemMenu *menu = NULL;
int totalItensMenu = 0;

Pedido *pedidos = NULL;
int totalPedidos = 0;

void limparBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

const char* statusParaString(StatusPedido status) {
    switch (status) {
        case PENDENTE: return "PENDENTE";
        case EM_PREPARO: return "EM PREPARO";
        case PRONTO: return "PRONTO";
        case ENTREGUE: return "ENTREGUE";
        default: return "DESCONHECIDO";
    }
}

int lerInteiroValido(const char *mensagem) {
    int valor;
    while (1) {
        printf("%s", mensagem);
        if (scanf("%d", &valor) == 1) {
            limparBuffer();
            return valor;
        } else {
            printf("\nEntrada invalida. Digite um numero.\n");
            limparBuffer();
        }
    }
}

float lerFloatValido(const char *mensagem) {
    float valor;
    while (1) {
        printf("%s", mensagem);
        if (scanf("%f", &valor) == 1) {
            limparBuffer();
            return valor;
        } else {
            printf("\nEntrada invalida. Digite um numero.\n");
            limparBuffer();
        }
    }
}

void adicionarItemMenu() {
    printf("\033[H\033[J");
    menu = realloc(menu, (totalItensMenu + 1) * sizeof(ItemMenu));
    if (menu == NULL) {
        printf("\nErro ao alocar memoria.\n");
        return;
    }

    printf("===========================================\n");
    printf("\n\tCADASTRO DE MENU\n\n===========================================\n\n");

    printf("Nome do item: ");
    fgets(menu[totalItensMenu].nome, sizeof(menu[totalItensMenu].nome), stdin);
    menu[totalItensMenu].nome[strcspn(menu[totalItensMenu].nome, "\n")] = '\0';

    printf("Descricao: ");
    fgets(menu[totalItensMenu].descricao, sizeof(menu[totalItensMenu].descricao), stdin);
    menu[totalItensMenu].descricao[strcspn(menu[totalItensMenu].descricao, "\n")] = '\0';

    menu[totalItensMenu].preco = lerFloatValido("Preco: ");

    printf("\n===========================================\n");

    int categoria;
    do {
        categoria = lerInteiroValido("\n| 0-ENTRADA | 1-PRINCIPAL | 2-SOBREMESA | 3-BEBIDA |\n\nDigite a categoria: ");
        if (categoria < 0 || categoria > 3) {
            printf("\nCategoria invalida. Digite um numero entre 0 e 3.\n");
        }
    } while (categoria < 0 || categoria > 3);
    menu[totalItensMenu].categoria = (Categoria)categoria;

    totalItensMenu++;
    printf("Item adicionado ao menu!\n");
    printf("\nPressione Enter para continuar...");
    limparBuffer();
}

int i;

void listarMenu() {
    printf("\033[H\033[J");
    printf("===========================================\n");
    printf("\n\t\tMENU\n");
    printf("\n===========================================\n");

    ItemMenu *entradas[100];
    ItemMenu *principais[100];
    ItemMenu *sobremesas[100];
    ItemMenu *bebidas[100];

    int totalEntradas = 0, totalPrincipais = 0, totalSobremesas = 0, totalBebidas = 0;

    for (i = 0; i < totalItensMenu; i++) {
        switch (menu[i].categoria) {
            case ENTRADA:
                entradas[totalEntradas++] = &menu[i];
                break;
            case PRINCIPAL:
                principais[totalPrincipais++] = &menu[i];
                break;
            case SOBREMESA:
                sobremesas[totalSobremesas++] = &menu[i];
                break;
            case BEBIDA:
                bebidas[totalBebidas++] = &menu[i];
                break;
        }
    }

    printf("\n--------- ENTRADAS ---------\n\n");
    for (i = 0; i < totalEntradas; i++) {
        printf("%d | %s (%s) - R$%.2f\n", i, entradas[i]->nome, entradas[i]->descricao, entradas[i]->preco);
    }

    printf("\n--------- PRINCIPAIS ---------\n\n");
    for (i = 0; i < totalPrincipais; i++) {
        printf("%d | %s (%s) - R$%.2f\n", i + totalEntradas, principais[i]->nome, principais[i]->descricao, principais[i]->preco);
    }

    printf("\n--------- SOBREMESAS ---------\n\n");
    for (i = 0; i < totalSobremesas; i++) {
        printf("%d | %s (%s) - R$%.2f\n", i + totalEntradas + totalPrincipais, sobremesas[i]->nome, sobremesas[i]->descricao, sobremesas[i]->preco);
    }

    printf("\n--------- BEBIDAS ---------\n\n");
    for (i = 0; i < totalBebidas; i++) {
        printf("%d | %s (%s) - R$%.2f\n", i + totalEntradas + totalPrincipais + totalSobremesas, bebidas[i]->nome, bebidas[i]->descricao, bebidas[i]->preco);
    }
    printf("\nPressione Enter para continuar...");
    limparBuffer();
}

void criarPedido() {
    printf("\033[H\033[J");
    pedidos = realloc(pedidos, (totalPedidos + 1) * sizeof(Pedido));
    if (pedidos == NULL) {
        printf("\nErro ao alocar memoria.\n");
        return;
    }

    Pedido *novoPedido = &pedidos[totalPedidos];
    novoPedido->id = totalPedidos + 1;
    printf("===========================================\n\n\t\tNovo Pedido\n\n===========================================\n");
    printf("Nome do cliente: ");
    fgets(novoPedido->nomeCliente, sizeof(novoPedido->nomeCliente), stdin);
    novoPedido->nomeCliente[strcspn(novoPedido->nomeCliente, "\n")] = '\0';

    novoPedido->quantidadeItens = 0;
    novoPedido->itens = NULL;
    novoPedido->status = PENDENTE;

    int opcao;
    do {
        listarMenu();
        printf("\n===========================================\n");
        opcao = lerInteiroValido("Escolha um item do menu (ou -1 para finalizar): ");
        if (opcao >= 0 && opcao < totalItensMenu) {
            novoPedido->itens = realloc(novoPedido->itens, (novoPedido->quantidadeItens + 1) * sizeof(ItemMenu));
            printf("Item adicionado com sucesso!");
            Sleep(2000);
            if (novoPedido->itens == NULL) {
                printf("Erro ao alocar memoria.\n");
                return;
            }
            novoPedido->itens[novoPedido->quantidadeItens] = menu[opcao];
            novoPedido->quantidadeItens++;
        } else if (opcao != -1) {
            printf("Indice invalido.\n");
        }
    } while (opcao != -1);

    totalPedidos++;
    printf("Pedido criado com sucesso!\n");
    printf("Pressione Enter para continuar...");
    limparBuffer();
}

int j;

void listarPedidos() {
    printf("\033[H\033[J");
    printf("\n===========================================\n\n\t\t PEDIDOS \n\n===========================================\n");
    for (i = 0; i < totalPedidos; i++) {
        printf("Pedido %d - Cliente: %s\n", pedidos[i].id, pedidos[i].nomeCliente);
        printf("Itens:\n");
        for (j = 0; j < pedidos[i].quantidadeItens; j++) {
            printf("  - %s (%s) - R$%.2f\n", pedidos[i].itens[j].nome, pedidos[i].itens[j].descricao, pedidos[i].itens[j].preco);
        }
        printf("Status: %s\n", statusParaString(pedidos[i].status));
    }
    printf("===========================================\n");
    printf("Pressione Enter para continuar...");
    limparBuffer();
}

void atualizarStatusPedido() {
    printf("\033[H\033[J");
    printf("===========================================\n\n\t\t Atualizacao de status\n\n===========================================\n\n");

    for (i = 0; i < totalPedidos; i++) {
        printf("Id: %d - Cliente: %s\n", pedidos[i].id, pedidos[i].nomeCliente);
    };

    printf("\n\n===========================================\n");

    int id = lerInteiroValido("ID do pedido: ");
    printf("\n===========================================\n");
    for (i = 0; i < totalPedidos; i++) {
        if (pedidos[i].id == id) {
            int status;
            do {
                status = lerInteiroValido("\n| 0 - PENDENTE | 1 - PREPARANDO | 2 - PRONTO | 3 - ENTREGUE |\n Digite um numero: ");
                if (status < 0 || status > 3) {
                    printf("\nStatus invalido. Digite um numero entre 0 e 3.\n");
                }
            } while (status < 0 || status > 3);
            pedidos[i].status = (StatusPedido)status;
            printf("\nStatus atualizado!\n");
            printf("\nPressione Enter para continuar...");
            limparBuffer();
            return;
        }
    }

    printf("Pedido nao encontrado.\n");
    printf("Pressione Enter para continuar...");
    limparBuffer();
    getchar();
}

void editarPedido() {
    printf("\033[H\033[J");
    printf("===========================================\n\n\t\t Editar Pedido\n\n===========================================\n\n");
    
    for (i = 0; i < totalPedidos; i++) {
        printf("Id: %d - Cliente: %s\n", pedidos[i].id, pedidos[i].nomeCliente);
    };
    
    printf("\n===========================================\n");
    int id = lerInteiroValido("ID do pedido que deseja editar: ");
    printf("\n===========================================\n");

    for (i = 0; i < totalPedidos; i++) {
        if (pedidos[i].id == id) {
            int opcao;
            do {
                printf("\n======= EDITANDO PEDIDO [%d] =======\n", pedidos[i].id);
                printf("1 - Alterar nome do cliente\n");
                printf("2 - Adicionar item ao pedido\n");
                printf("3 - Remover item do pedido\n");
                printf("4 - Alterar status do pedido\n");
                printf("0 - Voltar ao menu principal\n");
                printf("\n====================================\n");
                opcao = lerInteiroValido("Digite o Numero Correspondente A Escolha Da Opcao: ");

                switch (opcao) {
                    case 1: {
                        printf("Novo nome do cliente: ");
                        fgets(pedidos[i].nomeCliente, sizeof(pedidos[i].nomeCliente), stdin);
                        pedidos[i].nomeCliente[strcspn(pedidos[i].nomeCliente, "\n")] = '\0';
                        printf("Nome do cliente atualizado!\n");
                        break;
                    }
                    case 2: {
                        listarMenu();
                        int itemIndex = lerInteiroValido("Escolha um item do menu para adicionar (ou -1 para cancelar): ");
                        if (itemIndex >= 0 && itemIndex < totalItensMenu) {
                            pedidos[i].itens = realloc(pedidos[i].itens, (pedidos[i].quantidadeItens + 1) * sizeof(ItemMenu));
                            if (pedidos[i].itens == NULL) {
                                printf("Erro ao alocar memoria.\n");
                                return;
                            }
                            pedidos[i].itens[pedidos[i].quantidadeItens] = menu[itemIndex];
                            pedidos[i].quantidadeItens++;
                            printf("Item adicionado ao pedido!\n");
                        } else if (itemIndex != -1) {
                            printf("Indice invalido.\n");
                        }
                        break;
                    }
                    case 3: {
                        if (pedidos[i].quantidadeItens == 0) {
                            printf("O pedido nao possui itens para remover.\n");
                            break;
                        }
                        printf("Itens do pedido:\n");
                        for (j = 0; j < pedidos[i].quantidadeItens; j++) {
                            printf("%d - %s (%s) - R$%.2f\n", j, pedidos[i].itens[j].nome, pedidos[i].itens[j].descricao, pedidos[i].itens[j].preco);
                        }
                        int removeIndex = lerInteiroValido("Escolha o indice do item para remover (ou -1 para cancelar): ");
                        if (removeIndex >= 0 && removeIndex < pedidos[i].quantidadeItens) {
                            for (j = removeIndex; j < pedidos[i].quantidadeItens - 1; j++) {
                                pedidos[i].itens[j] = pedidos[i].itens[j + 1];
                            }
                            pedidos[i].quantidadeItens--;
                            pedidos[i].itens = realloc(pedidos[i].itens, pedidos[i].quantidadeItens * sizeof(ItemMenu));
                            printf("Item removido do pedido!\n");
                        } else if (removeIndex != -1) {
                            printf("Indice invalido.\n");
                        }
                        break;
                    }
                    case 4: {
                        int status;
                        do {
                            status = lerInteiroValido("Novo status (0-PENDENTE, 1-EM_PREPARO, 2-PRONTO, 3-ENTREGUE): ");
                            if (status < 0 || status > 3) {
                                printf("Status invalido. Digite um numero entre 0 e 3.\n");
                            }
                        } while (status < 0 || status > 3);
                        pedidos[i].status = (StatusPedido)status;
                        printf("Status atualizado!\n");
                        break;
                    }
                    case 0: {
                        printf("Retornando ao menu principal...\n");
                        break;
                    }
                    default: {
                        printf("Opcao invalida.\n");
                        break;
                    }
                }
                printf("Pressione Enter para continuar...");

                limparBuffer();
                system("cls");
            } while (opcao != 0);
            return;
        }
    }

    printf("Pedido nao encontrado.\n");
    printf("\nPressione Enter para continuar...");
    limparBuffer();
    getchar();
}

void editarItemMenu() {
    printf("\033[H\033[J");
    printf("===========================================\n\n\t\t Editar Item do Menu\n\n===========================================\n\n");

    if (totalItensMenu == 0) {
        printf("Nenhum item no menu para editar.\n");
        printf("Pressione Enter para continuar...");
        limparBuffer();
        return;
    }

    listarMenu();
    int itemIndex = lerInteiroValido("Escolha o indice do item que deseja editar (ou -1 para cancelar): ");

    if (itemIndex == -1) {
        printf("Edicao cancelada.\n");
        printf("Pressione Enter para continuar...");
        limparBuffer();
        return;
    }

    if (itemIndex < 0 || itemIndex >= totalItensMenu) {
        printf("Indice invalido.\n");
        printf("Pressione Enter para continuar...");
        limparBuffer();
        return;
    }

    ItemMenu *item = &menu[itemIndex];

    int opcao;
    do {
        printf("\n======= EDITANDO ITEM [%d] =======\n", itemIndex);
        printf("1 - Alterar nome\n");
        printf("2 - Alterar descricao\n");
        printf("3 - Alterar preco\n");
        printf("4 - Alterar categoria\n");
        printf("0 - Voltar ao menu principal\n");
        printf("\n====================================\n");
        opcao = lerInteiroValido("Digite o Numero Correspondente A Escolha Da Opcao: ");

        switch (opcao) {
            case 1: {
                printf("Novo nome do item: ");
                fgets(item->nome, sizeof(item->nome), stdin);
                item->nome[strcspn(item->nome, "\n")] = '\0';
                printf("Nome do item atualizado!\n");
                break;
            }
            case 2: {
                printf("Nova descricao do item: ");
                fgets(item->descricao, sizeof(item->descricao), stdin);
                item->descricao[strcspn(item->descricao, "\n")] = '\0';
                printf("Descricao do item atualizada!\n");
                break;
            }
            case 3: {
                item->preco = lerFloatValido("Novo preco do item: ");
                printf("Preco do item atualizado!\n");
                break;
            }
            case 4: {
                int categoria;
                do {
                    categoria = lerInteiroValido("\n| 0-ENTRADA | 1-PRINCIPAL | 2-SOBREMESA | 3-BEBIDA |\n\nDigite a nova categoria: ");
                    if (categoria < 0 || categoria > 3) {
                        printf("\nCategoria invalida. Digite um numero entre 0 e 3.\n");
                    }
                } while (categoria < 0 || categoria > 3);
                item->categoria = (Categoria)categoria;
                printf("Categoria do item atualizada!\n");
                break;
            }
            case 0: {
                printf("Retornando ao menu principal...\n");
                break;
            }
            default: {
                printf("Opcao invalida.\n");
                break;
            }
        }
        printf("Pressione Enter para continuar...");
        limparBuffer();
        system("cls");
    } while (opcao != 0);
}

int main() {
    setlocale(LC_ALL, "pt_BR.UTF-8");
    int opcao;
    do {
        printf("\033[H\033[J");
        printf("===========================================\n");
        printf("\n\tGERENCIAMENTO DE RESTAURANTE \n\n===========================================\n\n");
        printf(" 1 | Adicionar item ao menu\n");
        printf(" 2 | Listar menu\n");
        printf(" 3 | Criar pedido\n");
        printf(" 4 | Listar pedidos\n");
        printf(" 5 | Atualizar status do pedido\n");
        printf(" 6 | Editar pedido\n");
        printf(" 7 | Editar item do menu\n");
        printf(" 0 | encerrar\n");
        printf("\n===========================================\n");
        opcao = lerInteiroValido("Escolha uma opcao (Apenas numeros): ");

        switch (opcao) {
            case 1:
                adicionarItemMenu();
                break;
            case 2:
                listarMenu();
                break;
            case 3:
                criarPedido();
                break;
            case 4:
                listarPedidos();
                break;
            case 5:
                atualizarStatusPedido();
                break;
            case 6:
                editarPedido();
                break;
            case 7:  // Nova opção
                editarItemMenu();
                break;
            case 0:
                printf("Saindo...\n");
                break;
            default:
                printf("Opcao invalida.\n");
                printf("Pressione Enter para continuar...");
                limparBuffer();
                break;
        }
    } while (opcao != 0);

    if (menu != NULL) {
        free(menu);
    }
    for (i = 0; i < totalPedidos; i++) {
        if (pedidos[i].itens != NULL) {
            free(pedidos[i].itens);
        }
    }
    if (pedidos != NULL) {
        free(pedidos);
    }

    return 0;
}# Projeto_ED1
