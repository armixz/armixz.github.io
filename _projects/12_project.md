---
layout: page
title: Custom Shell Command Processor
description: Comprehensive Bash command-line interface in Unix environment
img: assets/img/12.jpg
importance: 12
category: work
github: https://github.com/armixz/Shell-Command-Processor
---

## Project Overview

Developed a **comprehensive Bash command-line interface application** in a Unix environment using shell scripting and system programming in **Fall 2018**. The implementation features custom command parsing, input validation, and process management capabilities, enhancing system interaction efficiency by streamlining command processing, automating task execution, and improving user experience for routine administrative operations.

## Shell Architecture Design

### Core Components
- **Command Parser**: Tokenization and syntax analysis
- **Process Manager**: Fork/exec process creation and management
- **Built-in Commands**: Internal command implementations
- **I/O Redirection**: File input/output handling
- **Pipeline Support**: Command chaining with pipes
- **Job Control**: Background/foreground process management

### System Integration
```
User Input → Command Parser → Process Manager → System Calls
     ↑             ↓              ↓              ↓
  Prompt    Built-in Check   Fork/Exec      Kernel
Generator      ↓              ↓              ↓
     ↑      External Cmd   Wait/Signal   Process
  Output ← I/O Redirection ← Job Control ← Execution
```

## Command Parser Implementation

### Lexical Analysis
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <fcntl.h>

#define MAX_CMD_LEN 1024
#define MAX_ARGS 64
#define MAX_JOBS 64

typedef enum {
    TOKEN_WORD,
    TOKEN_PIPE,
    TOKEN_REDIRECT_IN,
    TOKEN_REDIRECT_OUT,
    TOKEN_REDIRECT_APPEND,
    TOKEN_BACKGROUND,
    TOKEN_EOF
} token_type_t;

typedef struct {
    token_type_t type;
    char* value;
} token_t;

typedef struct {
    char** args;
    int argc;
    char* input_file;
    char* output_file;
    int append_output;
    int background;
} command_t;

// Tokenize input string
token_t* tokenize(char* input, int* token_count) {
    token_t* tokens = malloc(MAX_ARGS * sizeof(token_t));
    char* token_str;
    int count = 0;
    
    // Handle special characters and operators
    char* special_chars = "|<>&";
    char* current = input;
    
    while (*current && count < MAX_ARGS - 1) {
        // Skip whitespace
        while (*current == ' ' || *current == '\t') current++;
        if (*current == '\0') break;
        
        tokens[count].value = malloc(256);
        
        if (*current == '|') {
            tokens[count].type = TOKEN_PIPE;
            strcpy(tokens[count].value, "|");
            current++;
        } else if (*current == '<') {
            tokens[count].type = TOKEN_REDIRECT_IN;
            strcpy(tokens[count].value, "<");
            current++;
        } else if (*current == '>') {
            if (*(current + 1) == '>') {
                tokens[count].type = TOKEN_REDIRECT_APPEND;
                strcpy(tokens[count].value, ">>");
                current += 2;
            } else {
                tokens[count].type = TOKEN_REDIRECT_OUT;
                strcpy(tokens[count].value, ">");
                current++;
            }
        } else if (*current == '&') {
            tokens[count].type = TOKEN_BACKGROUND;
            strcpy(tokens[count].value, "&");
            current++;
        } else {
            // Regular word token
            tokens[count].type = TOKEN_WORD;
            int pos = 0;
            
            // Handle quoted strings
            if (*current == '"' || *current == '\'') {
                char quote = *current++;
                while (*current && *current != quote && pos < 255) {
                    tokens[count].value[pos++] = *current++;
                }
                if (*current == quote) current++; // Skip closing quote
            } else {
                // Regular word
                while (*current && *current != ' ' && *current != '\t' && 
                       !strchr(special_chars, *current) && pos < 255) {
                    tokens[count].value[pos++] = *current++;
                }
            }
            tokens[count].value[pos] = '\0';
        }
        count++;
    }
    
    // Add EOF token
    tokens[count].type = TOKEN_EOF;
    tokens[count].value = NULL;
    
    *token_count = count;
    return tokens;
}
```

### Command Structure Parsing
```c
// Parse tokens into command structure
command_t* parse_command(token_t* tokens, int* cmd_count) {
    command_t* commands = malloc(MAX_ARGS * sizeof(command_t));
    int current_cmd = 0;
    int token_idx = 0;
    
    // Initialize first command
    commands[current_cmd].args = malloc(MAX_ARGS * sizeof(char*));
    commands[current_cmd].argc = 0;
    commands[current_cmd].input_file = NULL;
    commands[current_cmd].output_file = NULL;
    commands[current_cmd].append_output = 0;
    commands[current_cmd].background = 0;
    
    while (tokens[token_idx].type != TOKEN_EOF) {
        switch (tokens[token_idx].type) {
            case TOKEN_WORD:
                commands[current_cmd].args[commands[current_cmd].argc] = 
                    strdup(tokens[token_idx].value);
                commands[current_cmd].argc++;
                break;
                
            case TOKEN_PIPE:
                // Null-terminate current command args
                commands[current_cmd].args[commands[current_cmd].argc] = NULL;
                
                // Start new command
                current_cmd++;
                commands[current_cmd].args = malloc(MAX_ARGS * sizeof(char*));
                commands[current_cmd].argc = 0;
                commands[current_cmd].input_file = NULL;
                commands[current_cmd].output_file = NULL;
                commands[current_cmd].append_output = 0;
                commands[current_cmd].background = 0;
                break;
                
            case TOKEN_REDIRECT_IN:
                token_idx++; // Move to filename
                if (tokens[token_idx].type == TOKEN_WORD) {
                    commands[current_cmd].input_file = strdup(tokens[token_idx].value);
                }
                break;
                
            case TOKEN_REDIRECT_OUT:
                token_idx++; // Move to filename
                if (tokens[token_idx].type == TOKEN_WORD) {
                    commands[current_cmd].output_file = strdup(tokens[token_idx].value);
                    commands[current_cmd].append_output = 0;
                }
                break;
                
            case TOKEN_REDIRECT_APPEND:
                token_idx++; // Move to filename
                if (tokens[token_idx].type == TOKEN_WORD) {
                    commands[current_cmd].output_file = strdup(tokens[token_idx].value);
                    commands[current_cmd].append_output = 1;
                }
                break;
                
            case TOKEN_BACKGROUND:
                commands[current_cmd].background = 1;
                break;
        }
        token_idx++;
    }
    
    // Null-terminate final command args
    commands[current_cmd].args[commands[current_cmd].argc] = NULL;
    
    *cmd_count = current_cmd + 1;
    return commands;
}
```

## Built-in Command Implementation

### Core Built-in Commands
```c
// Built-in command structure
typedef struct {
    char* name;
    int (*function)(char** args);
    char* description;
} builtin_t;

// Built-in command implementations
int builtin_cd(char** args) {
    if (args[1] == NULL) {
        // No argument, change to home directory
        char* home = getenv("HOME");
        if (home == NULL) {
            fprintf(stderr, "cd: HOME not set\n");
            return 1;
        }
        if (chdir(home) != 0) {
            perror("cd");
            return 1;
        }
    } else {
        if (chdir(args[1]) != 0) {
            perror("cd");
            return 1;
        }
    }
    return 0;
}

int builtin_pwd(char** args) {
    char cwd[1024];
    if (getcwd(cwd, sizeof(cwd)) != NULL) {
        printf("%s\n", cwd);
    } else {
        perror("pwd");
        return 1;
    }
    return 0;
}

int builtin_echo(char** args) {
    for (int i = 1; args[i] != NULL; i++) {
        printf("%s", args[i]);
        if (args[i + 1] != NULL) printf(" ");
    }
    printf("\n");
    return 0;
}

int builtin_export(char** args) {
    if (args[1] == NULL) {
        // Print all environment variables
        extern char** environ;
        for (int i = 0; environ[i] != NULL; i++) {
            printf("export %s\n", environ[i]);
        }
    } else {
        // Set environment variable
        char* eq_pos = strchr(args[1], '=');
        if (eq_pos != NULL) {
            *eq_pos = '\0';
            char* name = args[1];
            char* value = eq_pos + 1;
            
            if (setenv(name, value, 1) != 0) {
                perror("export");
                return 1;
            }
        } else {
            fprintf(stderr, "export: invalid format\n");
            return 1;
        }
    }
    return 0;
}

int builtin_history(char** args) {
    extern char** command_history;
    extern int history_count;
    
    for (int i = 0; i < history_count; i++) {
        printf("%4d  %s\n", i + 1, command_history[i]);
    }
    return 0;
}

int builtin_jobs(char** args) {
    extern job_t jobs[];
    extern int job_count;
    
    for (int i = 0; i < job_count; i++) {
        if (jobs[i].active) {
            printf("[%d]  %s  %s\n", i + 1, 
                   jobs[i].background ? "Running" : "Stopped",
                   jobs[i].command);
        }
    }
    return 0;
}

int builtin_exit(char** args) {
    // Clean up and exit
    cleanup_shell();
    exit(0);
}

// Built-in command table
builtin_t builtins[] = {
    {"cd", builtin_cd, "Change directory"},
    {"pwd", builtin_pwd, "Print working directory"},
    {"echo", builtin_echo, "Display text"},
    {"export", builtin_export, "Set environment variable"},
    {"history", builtin_history, "Show command history"},
    {"jobs", builtin_jobs, "List active jobs"},
    {"exit", builtin_exit, "Exit shell"},
    {NULL, NULL, NULL}
};

// Check if command is built-in
int execute_builtin(char** args) {
    if (args[0] == NULL) return 0;
    
    for (int i = 0; builtins[i].name != NULL; i++) {
        if (strcmp(args[0], builtins[i].name) == 0) {
            return builtins[i].function(args);
        }
    }
    return -1; // Not a built-in command
}
```

## Process Management

### Fork/Exec Implementation
```c
// Job control structure
typedef struct {
    pid_t pid;
    char* command;
    int background;
    int active;
    int job_id;
} job_t;

job_t jobs[MAX_JOBS];
int job_count = 0;

// Execute external command
int execute_external(command_t* cmd) {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        
        // Handle input redirection
        if (cmd->input_file) {
            int fd = open(cmd->input_file, O_RDONLY);
            if (fd < 0) {
                perror("input redirection");
                exit(1);
            }
            dup2(fd, STDIN_FILENO);
            close(fd);
        }
        
        // Handle output redirection
        if (cmd->output_file) {
            int flags = O_WRONLY | O_CREAT;
            flags |= cmd->append_output ? O_APPEND : O_TRUNC;
            
            int fd = open(cmd->output_file, flags, 0644);
            if (fd < 0) {
                perror("output redirection");
                exit(1);
            }
            dup2(fd, STDOUT_FILENO);
            close(fd);
        }
        
        // Execute command
        if (execvp(cmd->args[0], cmd->args) < 0) {
            perror(cmd->args[0]);
            exit(1);
        }
    } else if (pid < 0) {
        perror("fork");
        return 1;
    } else {
        // Parent process
        if (!cmd->background) {
            // Foreground job - wait for completion
            int status;
            waitpid(pid, &status, 0);
            return WEXITSTATUS(status);
        } else {
            // Background job - add to job list
            add_job(pid, create_command_string(cmd), 1);
            printf("[%d] %d\n", job_count, pid);
            return 0;
        }
    }
}
```

### Pipeline Implementation
```c
// Execute pipeline of commands
int execute_pipeline(command_t* commands, int cmd_count) {
    int pipefds[cmd_count - 1][2];
    pid_t pids[cmd_count];
    
    // Create pipes
    for (int i = 0; i < cmd_count - 1; i++) {
        if (pipe(pipefds[i]) < 0) {
            perror("pipe");
            return 1;
        }
    }
    
    // Execute each command in pipeline
    for (int i = 0; i < cmd_count; i++) {
        pids[i] = fork();
        
        if (pids[i] == 0) {
            // Child process
            
            // Set up input (from previous pipe or file)
            if (i > 0) {
                dup2(pipefds[i-1][0], STDIN_FILENO);
            } else if (commands[i].input_file) {
                int fd = open(commands[i].input_file, O_RDONLY);
                if (fd < 0) {
                    perror("input redirection");
                    exit(1);
                }
                dup2(fd, STDIN_FILENO);
                close(fd);
            }
            
            // Set up output (to next pipe or file)
            if (i < cmd_count - 1) {
                dup2(pipefds[i][1], STDOUT_FILENO);
            } else if (commands[i].output_file) {
                int flags = O_WRONLY | O_CREAT;
                flags |= commands[i].append_output ? O_APPEND : O_TRUNC;
                
                int fd = open(commands[i].output_file, flags, 0644);
                if (fd < 0) {
                    perror("output redirection");
                    exit(1);
                }
                dup2(fd, STDOUT_FILENO);
                close(fd);
            }
            
            // Close all pipe file descriptors
            for (int j = 0; j < cmd_count - 1; j++) {
                close(pipefds[j][0]);
                close(pipefds[j][1]);
            }
            
            // Execute command
            if (execvp(commands[i].args[0], commands[i].args) < 0) {
                perror(commands[i].args[0]);
                exit(1);
            }
        } else if (pids[i] < 0) {
            perror("fork");
            return 1;
        }
    }
    
    // Close all pipe file descriptors in parent
    for (int i = 0; i < cmd_count - 1; i++) {
        close(pipefds[i][0]);
        close(pipefds[i][1]);
    }
    
    // Wait for all processes (if foreground)
    if (!commands[cmd_count-1].background) {
        int status;
        for (int i = 0; i < cmd_count; i++) {
            waitpid(pids[i], &status, 0);
        }
        return WEXITSTATUS(status);
    } else {
        // Add pipeline as background job
        char* pipeline_cmd = create_pipeline_string(commands, cmd_count);
        add_job(pids[cmd_count-1], pipeline_cmd, 1);
        printf("[%d] %d\n", job_count, pids[cmd_count-1]);
        return 0;
    }
}
```

## Advanced Features

### Command History
```c
#define HISTORY_SIZE 1000

char* command_history[HISTORY_SIZE];
int history_count = 0;
int history_index = 0;

// Add command to history
void add_to_history(char* command) {
    if (history_count < HISTORY_SIZE) {
        command_history[history_count] = strdup(command);
        history_count++;
    } else {
        // Circular buffer - overwrite oldest
        free(command_history[history_index]);
        command_history[history_index] = strdup(command);
        history_index = (history_index + 1) % HISTORY_SIZE;
    }
}

// History expansion (!n, !!, !string)
char* expand_history(char* input) {
    if (input[0] != '!') return strdup(input);
    
    if (input[1] == '!') {
        // !! - repeat last command
        if (history_count > 0) {
            int last_idx = (history_count - 1) % HISTORY_SIZE;
            return strdup(command_history[last_idx]);
        }
    } else if (isdigit(input[1])) {
        // !n - repeat command n
        int n = atoi(&input[1]);
        if (n > 0 && n <= history_count) {
            return strdup(command_history[n - 1]);
        }
    } else {
        // !string - repeat last command starting with string
        char* search = &input[1];
        for (int i = history_count - 1; i >= 0; i--) {
            if (strncmp(command_history[i], search, strlen(search)) == 0) {
                return strdup(command_history[i]);
            }
        }
    }
    
    return strdup(input); // No expansion found
}
```

### Tab Completion
```c
#include <dirent.h>
#include <glob.h>

// Simple tab completion for files and commands
char** complete_command(char* partial, int* count) {
    char** completions = malloc(256 * sizeof(char*));
    *count = 0;
    
    // Check if it's a path completion
    if (strchr(partial, '/') != NULL) {
        // File/directory completion
        char* dir_part = strdup(partial);
        char* base_part = strrchr(dir_part, '/');
        
        if (base_part) {
            *base_part = '\0';
            base_part++;
            
            DIR* dir = opendir(dir_part[0] ? dir_part : ".");
            if (dir) {
                struct dirent* entry;
                while ((entry = readdir(dir)) != NULL && *count < 255) {
                    if (strncmp(entry->d_name, base_part, strlen(base_part)) == 0) {
                        completions[*count] = malloc(strlen(dir_part) + strlen(entry->d_name) + 2);
                        sprintf(completions[*count], "%s/%s", dir_part, entry->d_name);
                        (*count)++;
                    }
                }
                closedir(dir);
            }
        }
        free(dir_part);
    } else {
        // Command completion from PATH
        char* path = getenv("PATH");
        char* path_copy = strdup(path);
        char* dir = strtok(path_copy, ":");
        
        while (dir && *count < 255) {
            DIR* d = opendir(dir);
            if (d) {
                struct dirent* entry;
                while ((entry = readdir(d)) != NULL && *count < 255) {
                    if (strncmp(entry->d_name, partial, strlen(partial)) == 0) {
                        completions[*count] = strdup(entry->d_name);
                        (*count)++;
                    }
                }
                closedir(d);
            }
            dir = strtok(NULL, ":");
        }
        free(path_copy);
    }
    
    return completions;
}
```

## Main Shell Loop

### Interactive Shell
```c
int main() {
    char input[MAX_CMD_LEN];
    char* line;
    
    // Initialize shell
    init_shell();
    
    while (1) {
        // Display prompt
        display_prompt();
        
        // Read input
        if (fgets(input, sizeof(input), stdin) == NULL) {
            if (feof(stdin)) {
                printf("\n");
                break; // EOF (Ctrl+D)
            }
            continue;
        }
        
        // Remove newline
        input[strcspn(input, "\n")] = '\0';
        
        // Skip empty input
        if (strlen(input) == 0) continue;
        
        // Add to history
        add_to_history(input);
        
        // Expand history if needed
        char* expanded = expand_history(input);
        
        // Tokenize input
        int token_count;
        token_t* tokens = tokenize(expanded, &token_count);
        
        // Parse commands
        int cmd_count;
        command_t* commands = parse_command(tokens, &cmd_count);
        
        // Execute commands
        if (cmd_count == 1) {
            // Single command
            int builtin_result = execute_builtin(commands[0].args);
            if (builtin_result == -1) {
                // External command
                execute_external(&commands[0]);
            }
        } else if (cmd_count > 1) {
            // Pipeline
            execute_pipeline(commands, cmd_count);
        }
        
        // Cleanup
        free_tokens(tokens, token_count);
        free_commands(commands, cmd_count);
        free(expanded);
        
        // Check for completed background jobs
        check_background_jobs();
    }
    
    cleanup_shell();
    return 0;
}

void display_prompt() {
    char cwd[256];
    char* user = getenv("USER");
    char hostname[64];
    
    getcwd(cwd, sizeof(cwd));
    gethostname(hostname, sizeof(hostname));
    
    // Colorized prompt: user@hostname:cwd$
    printf("\033[32m%s\033[0m@\033[34m%s\033[0m:\033[33m%s\033[0m$ ", 
           user ? user : "user", hostname, cwd);
    fflush(stdout);
}
```

## Key Achievements

### Shell Functionality
- **Complete Command Processing**: Parsing, validation, and execution
- **Built-in Commands**: 7 essential shell built-ins implemented
- **I/O Redirection**: Full support for <, >, >> operators
- **Pipeline Support**: Multi-command pipelines with proper process handling
- **Job Control**: Background/foreground job management

### System Programming Excellence
- **Process Management**: Fork/exec model implementation
- **Signal Handling**: Proper signal management for job control
- **Memory Management**: Dynamic allocation with proper cleanup
- **Error Handling**: Comprehensive error detection and reporting

### User Experience Enhancement
- **Command History**: Full history with expansion support
- **Tab Completion**: File and command completion
- **Colorized Prompt**: Visual feedback with current directory
- **Administrative Automation**: Streamlined routine operations

## Technologies Used

- **C Programming** for system-level shell implementation
- **Unix System Calls** for process and file management
- **POSIX APIs** for portable system programming
- **Shell Scripting** for automation and configuration
- **GNU Readline** for advanced input handling (optional enhancement)

The project demonstrates comprehensive understanding of operating system concepts, system programming, process management, and Unix shell design essential for systems administration, DevOps engineering, and low-level software development. 