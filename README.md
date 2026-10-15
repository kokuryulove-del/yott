# yott

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define DICE_COUNT 5
#define CATEGORY_COUNT 12

// カテゴリ名
const char *category_names[CATEGORY_COUNT] = {
    "ONES",
    "TWOS",
    "THREES",
    "FOURS",
    "FIVES",
    "SIXES",
    "CHOICE",
    "FOUR OF A KIND",
    "FULL HOUSE",
    "SMALL STRAIGHT",
    "LARGE STRAIGHT",
    "YACHT"
};

// サイコロを振る
void roll_dice(int dice[], int keep[]) {
    for (int i = 0; i < DICE_COUNT; i++) {
        if (!keep[i]) {
            dice[i] = rand() % 6 + 1;
        }
    }
}

// サイコロ表示
void show_dice(int dice[]) {
    printf("\nサイコロ：");

    for (int i = 0; i < DICE_COUNT; i++) {
        printf("[%d] ", dice[i]);
    }

    printf("\n");
}

// 並び替え
void sort_dice(int dice[]) {
    for (int i = 0; i < DICE_COUNT - 1; i++) {
        for (int j = i + 1; j < DICE_COUNT; j++) {
            if (dice[i] > dice[j]) {
                int temp = dice[i];
                dice[i] = dice[j];
                dice[j] = temp;
            }
        }
    }
}

// 同じ数字の個数
int count_number(int dice[], int number) {
    int count = 0;

    for (int i = 0; i < DICE_COUNT; i++) {
        if (dice[i] == number) {
            count++;
        }
    }

    return count;
}

// 得点計算
int calculate_score(int dice[], int category) {

    int score = 0;

    // ONE ～ SIX
    if (category >= 0 && category <= 5) {
        int number = category + 1;

        for (int i = 0; i < DICE_COUNT; i++) {
            if (dice[i] == number) {
                score += number;
            }
        }
    }

    // CHOICE
    else if (category == 6) {
        for (int i = 0; i < DICE_COUNT; i++) {
            score += dice[i];
        }
    }

    // FOUR OF A KIND
    else if (category == 7) {
        for (int n = 1; n <= 6; n++) {
            if (count_number(dice, n) >= 4) {
                for (int i = 0; i < DICE_COUNT; i++) {
                    score += dice[i];
                }
                break;
            }
        }
    }

    // FULL HOUSE
    else if (category == 8) {
        int has_three = 0;
        int has_two = 0;

        for (int n = 1; n <= 6; n++) {
            int count = count_number(dice, n);

            if (count == 3) {
                has_three = 1;
            }

            if (count == 2) {
                has_two = 1;
            }
        }

        if (has_three && has_two) {
            score = 25;
        }
    }

    // SMALL STRAIGHT
    else if (category == 9) {
        int found = 0;

        for (int start = 1; start <= 3; start++) {
            int ok = 1;

            for (int n = start; n < start + 4; n++) {
                if (count_number(dice, n) == 0) {
                    ok = 0;
                }
            }

            if (ok) {
                found = 1;
            }
        }

        if (found) {
            score = 15;
        }
    }

    // LARGE STRAIGHT
    else if (category == 10) {
        sort_dice(dice);

        if (
            (dice[0] == 1 &&
             dice[1] == 2 &&
             dice[2] == 3 &&
             dice[3] == 4 &&
             dice[4] == 5)
            ||
            (dice[0] == 2 &&
             dice[1] == 3 &&
             dice[2] == 4 &&
             dice[3] == 5 &&
             dice[4] == 6)
        ) {
            score = 30;
        }
    }

    // YACHT
    else if (category == 11) {
        if (count_number(dice, dice[0]) == 5) {
            score = 50;
        }
    }

    return score;
}

// スコア表表示
void show_score(int score[2][CATEGORY_COUNT]) {

    printf("\n========== SCORE ==========\n");

    printf("カテゴリ              P1    P2\n");

    for (int i = 0; i < CATEGORY_COUNT; i++) {

        printf("%-20s ", category_names[i]);

        for (int p = 0; p < 2; p++) {

            if (score[p][i] == -1) {
                printf(" --   ");
            } else {
                printf("%3d   ", score[p][i]);
            }
        }

        printf("\n");
    }

    printf("===========================\n");
}

// 未使用カテゴリから選択
int select_category(int score[], int dice[]) {

    int category;

    while (1) {

        printf("\nカテゴリを選択してください\n");

        for (int i = 0; i < CATEGORY_COUNT; i++) {
            if (score[i] == -1) {
                printf("%2d : %s → %d点\n",
                       i + 1,
                       category_names[i],
                       calculate_score(dice, i));
            }
        }

        printf("番号 > ");
        scanf("%d", &category);

        category--;

        if (category >= 0 &&
            category < CATEGORY_COUNT &&
            score[category] == -1) {

            return category;
        }

        printf("そのカテゴリは選択できません。\n");
    }
}

// 1人分のターン
void play_turn(int player, int score[]) {

    int dice[DICE_COUNT];
    int keep[DICE_COUNT] = {0};

    printf("\n\n============================\n");
    printf("        PLAYER %d TURN\n", player + 1);
    printf("============================\n");

    // 最初のロール
    roll_dice(dice, keep);

    for (int roll = 1; roll <= 3; roll++) {

        printf("\n%d回目のロール", roll);

        show_dice(dice);

        if (roll == 3) {
            break;
        }

        printf("\n残したいサイコロを入力してください。\n");
        printf("例：1 3 5\n");
        printf("0 = 全部振り直す\n");

        printf("> ");

        int input;

        // 一旦keepをリセット
        for (int i = 0; i < DICE_COUNT; i++) {
            keep[i] = 0;
        }

        while (scanf("%d", &input) == 1 && input != 0) {

            if (input >= 1 && input <= 5) {
                keep[input - 1] = 1;
            }

            // 入力終了
            if (getchar() == '\n') {
                break;
            }
        }

        roll_dice(dice, keep);
    }

    // カテゴリ選択
    int category = select_category(score, dice);

    score[category] = calculate_score(dice, category);

    printf("\nPLAYER %d は %s を選択！\n",
           player + 1,
           category_names[category]);

    printf("獲得点数：%d点\n", score[category]);
}

int main(void) {

    int score[2][CATEGORY_COUNT];

    // -1 = 未使用
    for (int p = 0; p < 2; p++) {
        for (int i = 0; i < CATEGORY_COUNT; i++) {
            score[p][i] = -1;
        }
    }

    // 乱数初期化
    srand((unsigned int)time(NULL));

    // 12カテゴリ × 2人
    for (int round = 0; round < CATEGORY_COUNT; round++) {

        printf("\n\n");
        printf("******** ROUND %d ********\n", round + 1);

        play_turn(0, score);
        show_score(score);

        play_turn(1, score);
        show_score(score);
    }

    // 最終結果
    int total[2] = {0, 0};

    for (int p = 0; p < 2; p++) {
        for (int i = 0; i < CATEGORY_COUNT; i++) {
            total[p] += score[p][i];
        }
    }

    printf("\n\n");
    printf("================================\n");
    printf("          GAME OVER\n");
    printf("================================\n");

    printf("PLAYER 1 : %d点\n", total[0]);
    printf("PLAYER 2 : %d点\n", total[1]);

    if (total[0] > total[1]) {
        printf("\nPLAYER 1 WIN!\n");
    }
    else if (total[1] > total[0]) {
        printf("\nPLAYER 2 WIN!\n");
    }
    else {
        printf("\nDRAW!\n");
    }

    return 0;
}
