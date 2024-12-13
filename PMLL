#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include "unified_voice.h"
#include <sys/socket.h>
#include <arpa/inet.h>
#include "memory_silo.h"
#include "io_socket.h"

// Structure for PML Logic Loop
typedef struct {
  int id;
  int memory_silo_id;
  int io_socket_id;
  int free_c_present; // Flag indicating the presence of free.c
} pml_logic_loop_t;

pml_logic_loop_t* pml_logic_loop = NULL; // Global variable to maintain the state

// Function to initialize a socket
int init_socket(const char *ip, int port) {
    int sockfd;
    struct sockaddr_in server_addr;

    // Create socket
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) {
        perror("Socket creation failed");
        return -1;
    }

    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons(port);
    server_addr.sin_addr.s_addr = inet_addr(ip);

    // Connect to the server
    if (connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Connection failed");
        close(sockfd);
        return -1;
    }

    return sockfd;
}

// Initialization of the PML Logic Loop
void pml_logic_loop_init(int memory_silo_id, int io_socket_id) {
  pml_logic_loop = malloc(sizeof(pml_logic_loop_t));
  if (pml_logic_loop == NULL) {
    perror("Memory allocation for PML logic loop failed");
    exit(EXIT_FAILURE);
  }
  pml_logic_loop->id = 1;
  pml_logic_loop->memory_silo_id = memory_silo_id;
  pml_logic_loop->io_socket_id = io_socket_id;
  pml_logic_loop->free_c_present = 0; // Initialize the flag to indicate absence
}

// Processing within the PML Logic Loop
void pml_logic_loop_process(int io_socket_id, void* buffer, int length) {
  if (pml_logic_loop == NULL) {
    fprintf(stderr, "Error: PML logic loop has not been initialized.\n");
    return;
  }
  
  while (1) {
    if (pml_logic_loop->free_c_present) {
      printf("Free.c is detected. Sending signal to the free logic loop...\n");
      int signal = 1; // Value to signal that the condition has been met
      if (write(io_socket_id, &signal, sizeof(signal)) < 0) {
        perror("Failed to write to the socket");
      }
      system("./free"); // Trigger the execution of free.c
      break; // Exit the loop after signaling
    } else {
      printf("I am grateful.\n");
      // Here, you can process the buffer as needed
    }
  }
}

// Function to Retrieve the PML Logic Loop
pml_logic_loop_t* get_pml_logic_loop(int io_socket_id) {
  if (pml_logic_loop == NULL) {
    pml_logic_loop = malloc(sizeof(pml_logic_loop_t));
    if (pml_logic_loop == NULL) {
      perror("Memory allocation for PML logic loop failed");
      exit(EXIT_FAILURE);
    }
    pml_logic_loop->id = 1;
    pml_logic_loop->memory_silo_id = 1;
    pml_logic_loop->io_socket_id = io_socket_id;
    pml_logic_loop->free_c_present = 1; // Set the flag for demonstration
  }
  return pml_logic_loop;
}

// Function to cleanup the memory and socket
void pml_logic_loop_cleanup() {
  if (pml_logic_loop != NULL) {
    close(pml_logic_loop->io_socket_id); // Close the socket
    free(pml_logic_loop); // Free the memory allocated for the PML logic loop
    pml_logic_loop = NULL;
  }
}

// Main program (example usage)
int main() {
  int socket_id = init_socket("127.0.0.1", 8080); // Initialize socket with example IP and port
  if (socket_id < 0) {
    fprintf(stderr, "Socket initialization failed.\n");
    return EXIT_FAILURE;
  }

  // Initialize the PML logic loop with sample memory silo ID and socket ID
  pml_logic_loop_init(1, socket_id);
  
  // Example buffer (can be replaced with actual data)
  char buffer[1024] = "Example data";
  pml_logic_loop_process(socket_id, buffer, sizeof(buffer));
  
  // Cleanup before exit
  pml_logic_loop_cleanup();

  return EXIT_SUCCESS;
}
