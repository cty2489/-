# 使用指南和编译说明

## 🚀 快速开始

### 环境要求
- **开发环境**: Keil MDK-ARM 5.29 或更高版本
- **目标芯片**: STM32F103C8T6 (Cortex-M3, 64KB Flash, 20KB RAM)
- **调试器**: ST-Link V2 或兼容调试器
- **编译器**: ARM Compiler 5/6

### 硬件连接
```
STM32F103C8T6 最小系统板:
├── VCC  → 3.3V
├── GND  → 地线
├── PA9  → USART1_TX (串口输出)
├── PA10 → USART1_RX (串口输入)
├── SWDIO → ST-Link SWDIO
└── SWCLK → ST-Link SWCLK
```

## 📦 项目导入和配置

### 1. 导入项目
```bash
# 方法1: 直接打开项目文件
双击 Project.uvprojx

# 方法2: 通过Keil菜单
File → Open Project → 选择 Project.uvprojx
```

### 2. 项目配置检查
```c
// 在 User/stm32f10x_conf.h 中确认配置
#define USE_STDPERIPH_DRIVER
#define STM32F10X_MD        // 中等密度产品
#define HSE_VALUE 8000000   // 外部晶振频率
```

### 3. 算法版本选择
```c
// 在 User/main.c 中选择算法版本
#define USE_STATIC_ASTAR    // 静态内存版本(推荐)
// #define USE_DYNAMIC_ASTAR   // 动态内存版本(对比测试用)

// 测试模式选择
#define PERFORMANCE_TEST    // 性能测试模式
// #define MEMORY_TEST         // 内存测试模式
// #define SIMPLE_TEST         // 简单功能测试
```

## 🔧 编译和构建

### 编译步骤
```
1. Project → Rebuild all target files (Ctrl+F7)
2. 检查编译输出窗口是否有错误
3. 查看内存使用报告
```

### 编译输出分析
```
编译成功后查看:
├── Code Size: ~25KB (Flash使用)
├── RO-data:   ~2KB  (只读数据)
├── RW-data:   ~1KB  (可读写数据)
├── ZI-data:   ~6KB  (零初始化数据)
└── Total RAM: ~7KB  (总RAM使用)
```

### 优化设置
```
Target Options → C/C++ → Optimization:
├── Level: -O2 (Release) / -O0 (Debug)
├── Time: 勾选 (优化执行时间)
├── Space: 不勾选 (不优化代码大小)
└── Debug: 根据需要选择
```

## 🎯 使用示例

### 基本使用示例
```c
#include "a_star_static.h"
#include "systick.h"

int main(void)
{
    // 系统初始化
    SystemInit();
    SysTick_Init();
    USART1_Init(115200);
    
    // 定义测试迷宫 (20x20)
    uint8_t maze[20][20] = {
        {0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0},
        {0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0},
        // ... 更多迷宫数据
        {0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0}
    };
    
    // 设置起点和终点
    Point start = {1, 1};
    Point goal = {18, 18};
    
    // 执行A*算法
    uint32_t start_time = SysTick_GetTick();
    PathStatic path = a_star_static(maze, start, goal);
    uint32_t end_time = SysTick_GetTick();
    
    // 输出结果
    if (path.length > 0) {
        printf("路径找到! 长度: %d\n", path.length);
        printf("执行时间: %d 微秒\n", end_time - start_time);
        
        // 打印路径
        for (int i = 0; i < path.length; i++) {
            printf("(%d,%d) ", path.points[i].x, path.points[i].y);
        }
        printf("\n");
    } else {
        printf("未找到路径!\n");
    }
    
    while(1) {
        // 主循环
    }
}
```

### 内存监控示例
```c
#include "stm32_memory_monitor.h"

int main(void)
{
    // 初始化内存监控
    memory_monitor_init();
    
    // 开始内存测量
    memory_monitor_start();
    
    // 执行A*算法
    PathStatic path = a_star_static(maze, start, goal);
    
    // 结束内存测量
    memory_monitor_stop();
    
    // 获取内存统计
    MemoryStats stats = memory_monitor_get_stats();
    
    // 打印详细报告
    memory_monitor_print_stats();
    
    return 0;
}
```

### 性能测试示例
```c
#define TEST_ITERATIONS 100

void performance_test(void)
{
    uint32_t total_time = 0;
    uint32_t min_time = UINT32_MAX;
    uint32_t max_time = 0;
    
    printf("开始性能测试 (%d次迭代)...\n", TEST_ITERATIONS);
    
    for (int i = 0; i < TEST_ITERATIONS; i++) {
        uint32_t start = SysTick_GetTick();
        PathStatic path = a_star_static(test_maze, start_point, goal_point);
        uint32_t end = SysTick_GetTick();
        
        uint32_t duration = end - start;
        total_time += duration;
        
        if (duration < min_time) min_time = duration;
        if (duration > max_time) max_time = duration;
        
        printf("第%d次: %d μs\n", i+1, duration);
    }
    
    printf("\n性能统计:\n");
    printf("平均时间: %d μs\n", total_time / TEST_ITERATIONS);
    printf("最短时间: %d μs\n", min_time);
    printf("最长时间: %d μs\n", max_time);
}
```

## 🔍 调试和测试

### 调试配置
```
Debug Options:
├── Use: ST-Link Debugger
├── Settings → Flash Download:
│   ├── Reset and Run: 勾选
│   ├── Download Function: 勾选
│   └── Verify: 勾选
└── Trace → Enable: 根据需要
```

### 串口调试
```c
// 串口配置 (115200, 8N1)
USART1_Init(115200);

// 调试输出
printf("A* Algorithm Test\n");
printf("Maze Size: %dx%d\n", MAZE_WIDTH, MAZE_HEIGHT);
printf("Start: (%d,%d), Goal: (%d,%d)\n", 
       start.x, start.y, goal.x, goal.y);
```

### 断点调试
```
推荐断点位置:
├── a_star_static() 函数入口
├── find_path() 主循环
├── add_to_open_list() 节点添加
├── get_lowest_f_node() 节点选择
└── reconstruct_path() 路径重建
```

## 📊 性能分析工具

### 1. 内存使用分析
```c
// 使用内置内存监控工具
#include "stm32_memory_monitor.h"

// 获取静态A*内存使用
uint32_t static_memory = get_static_astar_memory_size();
printf("静态内存使用: %d bytes\n", static_memory);
```

### 2. 执行时间测量
```c
// 高精度时间测量
uint32_t start_tick = SysTick_GetTick();
// ... 执行算法
uint32_t end_tick = SysTick_GetTick();
uint32_t duration_us = end_tick - start_tick;
```

### 3. 栈使用监控
```c
// 栈使用情况检查
extern uint32_t __initial_sp;
uint32_t current_sp = __get_MSP();
uint32_t stack_used = __initial_sp - current_sp;
printf("栈使用: %d bytes\n", stack_used);
```

## 🛠️ 常见问题和解决方案

### 编译错误
```
错误: "identifier not declared"
解决: 检查头文件包含和函数声明

错误: "undefined reference"
解决: 检查源文件是否添加到项目中

错误: "region overflowed"
解决: 检查内存使用，可能需要优化数据结构
```

### 运行时问题
```
问题: 程序卡死或重启
解决: 检查栈溢出，增加栈大小或优化递归调用

问题: 路径不正确
解决: 检查迷宫数据和启发函数实现

问题: 性能不佳
解决: 启用编译器优化，检查算法参数
```

### 内存问题
```
问题: RAM使用过多
解决: 减少MAX_NODES或优化数据结构

问题: Flash空间不足
解决: 启用代码优化，移除不必要的功能

问题: 栈溢出
解决: 增加栈大小或减少递归深度
```

## 📈 性能优化建议

### 编译器优化
```c
// 使用内联函数减少函数调用开销
static inline uint32_t manhattan_distance(Point a, Point b) {
    return abs(a.x - b.x) + abs(a.y - b.y);
}

// 使用寄存器变量提高访问速度
register uint32_t i;
```

### 算法优化
```c
// 优化数据结构对齐
typedef struct __attribute__((packed)) {
    uint8_t x, y;           // 使用8位坐标
    uint16_t g, h, f;       // 使用16位代价值
} StaticNode;

// 使用位操作优化
#define SET_CLOSED(x, y) (closed_set[(y)*MAZE_WIDTH + (x)] = 1)
#define IS_CLOSED(x, y) (closed_set[(y)*MAZE_WIDTH + (x)])
```

### 内存优化
```c
// 减少全局变量使用
static AStarState g_state;  // 单例模式

// 优化数组大小
#define MAX_NODES 150       // 根据实际需求调整
#define MAX_PATH_LENGTH 100 // 根据迷宫大小调整
```

## 📝 开发最佳实践

### 代码规范
- 使用有意义的变量名
- 添加必要的注释
- 保持函数简洁
- 使用const修饰只读参数

### 测试策略
- 单元测试每个函数
- 集成测试完整流程
- 性能测试多种场景
- 边界条件测试

### 版本控制
- 提交前测试代码
- 写清楚提交信息
- 定期备份项目
- 标记重要版本

---

这个使用指南提供了从项目导入到性能优化的完整流程，帮助开发者快速上手并充分利用这个A*算法优化项目。
