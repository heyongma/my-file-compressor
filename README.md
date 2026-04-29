# my-file-compressor
A command-line file compressor and decompressor implementing the RLE algorithm in C.
/* Complete RLE File Compressor - Fixed Version */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 函数声明
void compress_file(const char* input_filename, const char* output_filename);
void decompress_file(const char* input_filename, const char* output_filename);
void print_help();
void clear_input_buffer();

int main() {
    int choice = 0;
    
    while (1) {
        system("cls");  // 清屏
        
        printf("=========================================\n");
        printf("         FILE COMPRESSOR 1.0\n");
        printf("=========================================\n");
        printf("1. Compress file\n");
        printf("2. Decompress file\n");
        printf("3. Help\n");
        printf("0. Exit\n");
        printf("=========================================\n");
        printf("Choose (0-3): ");
        
        if (scanf("%d", &choice) != 1) {
            clear_input_buffer();
            printf("\nPlease enter a number (0-3)!\n");
            continue;
        }
        clear_input_buffer();
        
        switch (choice) {
            case 1: {
                char input[100], output[100];
                
                printf("\n--- Compress File ---\n");
                printf("Enter source filename: ");
                scanf("%99s", input);
                clear_input_buffer();
                
                printf("Enter compressed filename: ");
                scanf("%99s", output);
                clear_input_buffer();
                
                compress_file(input, output);
                break;
            }
            
            case 2: {
                char input[100], output[100];
                
                printf("\n--- Decompress File ---\n");
                printf("Enter compressed filename: ");
                scanf("%99s", input);
                clear_input_buffer();
                
                printf("Enter decompressed filename: ");
                scanf("%99s", output);
                clear_input_buffer();
                
                decompress_file(input, output);
                break;
            }
            
            case 3:
                print_help();
                break;
                
            case 0:
                printf("\nGoodbye!\n");
                return 0;
                
            default:
                printf("\nInvalid choice! Please enter 0-3.\n");
        }
    }
    
    return 0;
}

// 清空输入缓冲区
void clear_input_buffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

// 压缩函数 - 修复版本
void compress_file(const char* input_filename, const char* output_filename) {
    printf("\n[INFO] Compressing '%s' to '%s'...\n", input_filename, output_filename);
    
    // 1. 打开源文件
    FILE* in = fopen(input_filename, "rb");
    if (in == NULL) {
        printf("[ERROR] Cannot open source file: %s\n", input_filename);
        printf("Make sure file exists and name is correct.\n");
        return;
    }
    
    // 2. 创建目标文件
    FILE* out = fopen(output_filename, "wb");
    if (out == NULL) {
        printf("[ERROR] Cannot create target file: %s\n", output_filename);
        fclose(in);
        return;
    }
    
    int current_char, next_char;
    int count = 1;
    int original_size = 0;
    int compressed_size = 0;
    
    // 3. 读取第一个字符
    current_char = fgetc(in);
    
    if (current_char == EOF) {
        printf("[WARNING] Source file is empty.\n");
        fclose(in);
        fclose(out);
        return;
    }
    
    // 4. 压缩循环 - 完全重写的逻辑
    while (1) {
        // 读取下一个字符
        next_char = fgetc(in);
        
        if (next_char == EOF) {
            // 文件结束，写入最后一组
            fputc(count, out);
            fputc(current_char, out);
            compressed_size += 2;
            original_size += count;  // 统计原始大小
            break;
        }
        
        if (current_char == next_char) {
            // 字符相同，增加计数
            count++;
        } else {
            // 字符不同，写入[计数][字符]
            fputc(count, out);
            fputc(current_char, out);
            compressed_size += 2;
            original_size += count;  // 统计原始大小
            
            // 重置计数器
            count = 1;
            current_char = next_char;
        }
    }
    
    // 5. 强制刷新缓冲区
    fflush(out);
    
    // 6. 关闭文件
    fclose(in);
    fclose(out);
    
    // 7. 验证文件是否生成
    printf("\n[VERIFY] Checking if file was created...\n");
    
    FILE* test = fopen(output_filename, "rb");
    if (test == NULL) {
        printf("[ERROR] File was not created: %s\n", output_filename);
    } else {
        fseek(test, 0, SEEK_END);
        long file_size = ftell(test);
        fclose(test);
        
        if (file_size > 0) {
            printf("[SUCCESS] File '%s' created (%ld bytes)\n", output_filename, file_size);
            
            // 显示文件内容的前20字节
            test = fopen(output_filename, "rb");
            if (test) {
                printf("First 20 bytes (hex): ");
                for (int i = 0; i < 20 && i < file_size; i++) {
                    int byte = fgetc(test);
                    if (byte != EOF) {
                        printf("%02X ", byte);
                    }
                }
                fclose(test);
                printf("\n");
            }
        } else {
            printf("[WARNING] File '%s' is empty!\n", output_filename);
        }
    }
    
    // 8. 显示压缩信息
    printf("\n--- Compression Complete ---\n");
    printf("Original size: %d bytes\n", original_size);
    printf("Compressed size: %d bytes\n", compressed_size);
    
    if (original_size > 0) {
        float ratio = (1.0 - (float)compressed_size / original_size) * 100;
        printf("Compression ratio: %.1f%%\n", ratio);
    }
    
    printf("\nPress Enter to continue...");
    clear_input_buffer();
    getchar();
}

// 解压函数
void decompress_file(const char* input_filename, const char* output_filename) {
    printf("\n[INFO] Decompressing '%s' to '%s'...\n", input_filename, output_filename);
    
    FILE* in = fopen(input_filename, "rb");
    if (in == NULL) {
        printf("[ERROR] Cannot open compressed file: %s\n", input_filename);
        return;
    }
    
    FILE* out = fopen(output_filename, "wb");
    if (out == NULL) {
        printf("[ERROR] Cannot create output file: %s\n", output_filename);
        fclose(in);
        return;
    }
    
    int count, value;
    int decompressed_size = 0;
    
    // 解压循环
    while (1) {
        // 读取[计数]
        count = fgetc(in);
        if (count == EOF) break;
        
        // 读取[字符值]
        value = fgetc(in);
        if (value == EOF) break;
        
        // 写入指定次数的字符
        for (int i = 0; i < count; i++) {
            fputc(value, out);
            decompressed_size++;
        }
    }
    
    fclose(in);
    fclose(out);
    
    printf("\n--- Decompression Complete ---\n");
    printf("Restored size: %d bytes\n", decompressed_size);
    printf("✓ File restored successfully!\n");
    
    printf("\nPress Enter to continue...");
    clear_input_buffer();
    getchar();
}

void print_help() {
    printf("\n=== File Compressor Help ===\n");
    printf("1. Compress: Uses RLE (Run-Length Encoding)\n");
    printf("2. Decompress: Restores original file\n");
    printf("3. Example: 'AAAABBB' -> 4A3B\n");
    printf("=============================\n");
    
    printf("\nPress Enter to continue...");
    clear_input_buffer();
    getchar();
}
