#include <SDL2/SDL.h>
#include <stdbool.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600

#define PADDLE_WIDTH 100
#define PADDLE_HEIGHT 15
#define BALL_SIZE 10

#define ROWS 7  // 무지개 7색
#define COLS 10
#define BLOCK_WIDTH 70
#define BLOCK_HEIGHT 20
#define BLOCK_PADDING 10
#define OFFSET_X 45
#define OFFSET_Y 50

// 무지개 색상 정의 (빨, 주, 노, 초, 파, 남, 보)
SDL_Color rainbowColors[7] = {
    {255, 0, 0, 255},     // 빨강
    {255, 127, 0, 255},   // 주황
    {255, 255, 0, 255},   // 노랑
    {0, 255, 0, 255},     // 초록
    {0, 0, 255, 255},     // 파랑
    {75, 0, 130, 255},    // 남색
    {143, 0, 255, 255}    // 보라
};

typedef struct {
    float x, y;
    float dx, dy;
} Ball;

typedef struct {
    float x, y;
} Paddle;

typedef struct {
    int x, y;
    bool active;
} Block;

int main(int argc, char* argv[]) {
    if (SDL_Init(SDL_INIT_VIDEO) < 0) {
        return 1;
    }

    SDL_Window* window = SDL_CreateWindow("Rainbow Breakout", 
        SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, 
        SCREEN_WIDTH, SCREEN_HEIGHT, SDL_WINDOW_SHOWN);
    
    SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);

    // 패들 초기화
    Paddle paddle = { SCREEN_WIDTH / 2 - PADDLE_WIDTH / 2, SCREEN_HEIGHT - 50 };
    
    // 공 초기화
    Ball ball = { SCREEN_WIDTH / 2, SCREEN_HEIGHT / 2, 4.0f, -4.0f };

    // 블록 초기화
    Block blocks[ROWS][COLS];
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            blocks[i][j].x = OFFSET_X + j * (BLOCK_WIDTH + BLOCK_PADDING);
            blocks[i][j].y = OFFSET_Y + i * (BLOCK_HEIGHT + BLOCK_PADDING);
            blocks[i][j].active = true;
        }
    }

    bool running = true;
    SDL_Event event;

    while (running) {
        // 이벤트 처리
        while (SDL_PollEvent(&event)) {
            if (event.type == SDL_QUIT) {
                running = false;
            }
        }

        // 키보드 입력 (좌우 이동)
        const Uint8* state = SDL_GetKeyboardState(NULL);
        if (state[SDL_SCANCODE_LEFT] && paddle.x > 0) {
            paddle.x -= 8.0f;
        }
        if (state[SDL_SCANCODE_RIGHT] && paddle.x < SCREEN_WIDTH - PADDLE_WIDTH) {
            paddle.x += 8.0f;
        }

        // 공 이동
        ball.x += ball.dx;
        ball.y += ball.dy;

        // 벽 충돌 (좌우)
        if (ball.x <= 0 || ball.x >= SCREEN_WIDTH - BALL_SIZE) {
            ball.dx = -ball.dx;
        }
        // 벽 충돌 (천장)
        if (ball.y <= 0) {
            ball.dy = -ball.dy;
        }
        // 바닥에 떨어지면 게임 오버 (위치 리셋)
        if (ball.y >= SCREEN_HEIGHT) {
            ball.x = SCREEN_WIDTH / 2;
            ball.y = SCREEN_HEIGHT / 2;
            ball.dy = -4.0f;
        }

        // 패들과 공 충돌
        if (ball.y + BALL_SIZE >= paddle.y && 
            ball.x + BALL_SIZE >= paddle.x && 
            ball.x <= paddle.x + PADDLE_WIDTH) {
            ball.dy = -ball.dy;
        }

        // 블록과 공 충돌 검사
        for (int i = 0; i < ROWS; i++) {
            for (int j = 0; j < COLS; j++) {
                if (blocks[i][j].active) {
                    if (ball.x + BALL_SIZE >= blocks[i][j].x &&
                        ball.x <= blocks[i][j].x + BLOCK_WIDTH &&
                        ball.y + BALL_SIZE >= blocks[i][j].y &&
                        ball.y <= blocks[i][j].y + BLOCK_HEIGHT) {
                        blocks[i][j].active = false;
                        ball.dy = -ball.dy;
                    }
                }
            }
        }

        // 화면 그리기 (렌더링)
        SDL_SetRenderDrawColor(renderer, 20, 20, 20, 255); // 배경색 (어두운 회색)
        SDL_RenderClear(renderer);

        // 패들 그리기 (흰색)
        SDL_SetRenderDrawColor(renderer, 255, 255, 255, 255);
        SDL_Rect paddleRect = { (int)paddle.x, (int)paddle.y, PADDLE_WIDTH, PADDLE_HEIGHT };
        SDL_RenderFillRect(renderer, &paddleRect);

        // 공 그리기 (노란색)
        SDL_SetRenderDrawColor(renderer, 255, 255, 0, 255);
        SDL_Rect ballRect = { (int)ball.x, (int)ball.y, BALL_SIZE, BALL_SIZE };
        SDL_RenderFillRect(renderer, &ballRect);

        // 무지개 블록 그리기 (행마다 다른 색상 적용)
        for (int i = 0; i < ROWS; i++) {
            SDL_SetRenderDrawColor(renderer, rainbowColors[i].r, rainbowColors[i].g, rainbowColors[i].b, 255);
            for (int j = 0; j < COLS; j++) {
                if (blocks[i][j].active) {
                    SDL_Rect blockRect = { blocks[i][j].x, blocks[i][j].y, BLOCK_WIDTH, BLOCK_HEIGHT };
                    SDL_RenderFillRect(renderer, &blockRect);
                }
            }
        }

        SDL_RenderPresent(renderer);
        SDL_Delay(16); // 약 60 FPS 유지
    }

    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();

    return 0;
}
