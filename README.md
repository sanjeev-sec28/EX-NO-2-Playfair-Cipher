## EX.NO:2—IMPLEMENTATION OF PLAYFAIR CIPHER

 

## AIM:
To write a C program to implement the Playfair Substitution technique.

## DESCRIPTION:

The Playfair cipher starts with creating a key table. The key table is a 5×5 grid of letters that will act as the key for encrypting your plaintext. Each of the 25 letters must be unique and one letter of the alphabet is omitted from the table (as there are 25 spots and 26 letters in the alphabet).

To encrypt a message, one would break the message into digrams (groups of 2 letters) such that, for example, "HelloWorld" becomes "HE LL OW OR LD", and map them out on the key table. The two letters of the diagram are considered as the opposite corners of a rectangle in the key table. Note the relative position of the corners of this rectangle. Then apply the following 4 rules, in order, to each pair of letters in the plaintext:

1.	If both letters are the same (or only one letter is left), add an "X" after the first letter
2.	If the letters appear on the same row of your table, replace them with the letters to their immediate right respectively
3.	If the letters appear on the same column of your table, replace them with the letters immediately below respectively
4.	If the letters are not on the same row or column, replace them with the letters on the same row respectively but at the other pair of corners of the rectangle defined by the original pair.

## EXAMPLE:
![image](https://github.com/Hemamanigandan/EX-NO-2-/assets/149653568/e6858d4f-b122-42ba-acdb-db18ec2e9675)

 

## ALGORITHM:

- STEP-1: Read the plain text from the user.
- STEP-2: Read the keyword from the user.
- STEP-3: Arrange the keyword without duplicates in a 5*5 matrix in the row order and fill the remaining cells with missed out letters in alphabetical order. Note that ‘i’ and ‘j’ takes the same cell.
- STEP-4: Group the plain text in pairs and match the corresponding corner letters by forming a rectangular grid.
- STEP-5: Display the obtained cipher text.




## Program:
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <stdbool.h>

#define SIZE 5

// Function prototypes
void prepareText(char *text, char *prepared);
void generateKeyTable(char *key, char keyTable[SIZE][SIZE]);
void findPosition(char keyTable[SIZE][SIZE], char ch, int *row, int *col);
void encryptPair(char keyTable[SIZE][SIZE], char a, char b, char *encrypted);
void decryptPair(char keyTable[SIZE][SIZE], char a, char b, char *decrypted);
void encrypt(char *plaintext, char *ciphertext, char keyTable[SIZE][SIZE]);
void decrypt(char *ciphertext, char *plaintext, char keyTable[SIZE][SIZE]);

int main() {
    char key[100], plaintext[500], ciphertext[500], decryptedtext[500];
    char keyTable[SIZE][SIZE];
    int choice;
    
    printf("=== PLAYFAIR CIPHER IMPLEMENTATION ===\n\n");
    
    // Get the key from user
    printf("Enter the key: ");
    fgets(key, sizeof(key), stdin);
    key[strcspn(key, "\n")] = 0; // Remove newline
    
    // Generate the key table
    generateKeyTable(key, keyTable);
    
    printf("\nGenerated Key Table (5x5):\n");
    for(int i = 0; i < SIZE; i++) {
        for(int j = 0; j < SIZE; j++) {
            printf("%c ", keyTable[i][j]);
        }
        printf("\n");
    }
    
    printf("\n1. Encrypt\n2. Decrypt\n");
    printf("Enter your choice (1 or 2): ");
    scanf("%d", &choice);
    getchar(); // Consume newline
    
    if(choice == 1) {
        printf("\nEnter the plaintext: ");
        fgets(plaintext, sizeof(plaintext), stdin);
        plaintext[strcspn(plaintext, "\n")] = 0;
        
        encrypt(plaintext, ciphertext, keyTable);
        printf("\nEncrypted text: %s\n", ciphertext);
    }
    else if(choice == 2) {
        printf("\nEnter the ciphertext: ");
        fgets(ciphertext, sizeof(ciphertext), stdin);
        ciphertext[strcspn(ciphertext, "\n")] = 0;
        
        decrypt(ciphertext, decryptedtext, keyTable);
        printf("\nDecrypted text: %s\n", decryptedtext);
    }
    else {
        printf("Invalid choice!\n");
    }
    
    return 0;
}

// Prepare the text (convert to uppercase, replace J with I, remove non-alphabetic)
void prepareText(char *text, char *prepared) {
    int i, j = 0;
    char ch;
    
    for(i = 0; text[i] != '\0'; i++) {
        ch = toupper(text[i]);
        
        if(ch >= 'A' && ch <= 'Z') {
            // Replace J with I
            if(ch == 'J')
                ch = 'I';
            prepared[j++] = ch;
        }
    }
    prepared[j] = '\0';
}

// Generate the 5x5 key table
void generateKeyTable(char *key, char keyTable[SIZE][SIZE]) {
    bool used[26] = {false};
    char preparedKey[200];
    int i, j, k = 0;
    char ch;
    
    // Prepare the key (remove duplicates, convert to uppercase)
    prepareText(key, preparedKey);
    
    // Mark 'J' as used (since we treat J as I)
    used['J' - 'A'] = true;
    
    // First, fill the key table with key letters
    for(i = 0; preparedKey[i] != '\0'; i++) {
        ch = preparedKey[i];
        int index = ch - 'A';
        
        if(!used[index]) {
            keyTable[k / SIZE][k % SIZE] = ch;
            used[index] = true;
            k++;
        }
    }
    
    // Then fill the remaining letters of alphabet
    for(ch = 'A'; ch <= 'Z'; ch++) {
        int index = ch - 'A';
        
        if(!used[index]) {
            keyTable[k / SIZE][k % SIZE] = ch;
            k++;
        }
    }
}

// Find position of a character in the key table
void findPosition(char keyTable[SIZE][SIZE], char ch, int *row, int *col) {
    int i, j;
    
    for(i = 0; i < SIZE; i++) {
        for(j = 0; j < SIZE; j++) {
            if(keyTable[i][j] == ch) {
                *row = i;
                *col = j;
                return;
            }
        }
    }
}

// Encrypt a pair of characters
void encryptPair(char keyTable[SIZE][SIZE], char a, char b, char *encrypted) {
    int row1, col1, row2, col2;
    
    findPosition(keyTable, a, &row1, &col1);
    findPosition(keyTable, b, &row2, &col2);
    
    if(row1 == row2) {
        // Same row - shift right
        encrypted[0] = keyTable[row1][(col1 + 1) % SIZE];
        encrypted[1] = keyTable[row2][(col2 + 1) % SIZE];
    }
    else if(col1 == col2) {
        // Same column - shift down
        encrypted[0] = keyTable[(row1 + 1) % SIZE][col1];
        encrypted[1] = keyTable[(row2 + 1) % SIZE][col2];
    }
    else {
        // Rectangle - swap columns
        encrypted[0] = keyTable[row1][col2];
        encrypted[1] = keyTable[row2][col1];
    }
}

// Decrypt a pair of characters
void decryptPair(char keyTable[SIZE][SIZE], char a, char b, char *decrypted) {
    int row1, col1, row2, col2;
    
    findPosition(keyTable, a, &row1, &col1);
    findPosition(keyTable, b, &row2, &col2);
    
    if(row1 == row2) {
        // Same row - shift left
        decrypted[0] = keyTable[row1][(col1 - 1 + SIZE) % SIZE];
        decrypted[1] = keyTable[row2][(col2 - 1 + SIZE) % SIZE];
    }
    else if(col1 == col2) {
        // Same column - shift up
        decrypted[0] = keyTable[(row1 - 1 + SIZE) % SIZE][col1];
        decrypted[1] = keyTable[(row2 - 1 + SIZE) % SIZE][col2];
    }
    else {
        // Rectangle - swap columns
        decrypted[0] = keyTable[row1][col2];
        decrypted[1] = keyTable[row2][col1];
    }
}

// Encrypt the plaintext
void encrypt(char *plaintext, char *ciphertext, char keyTable[SIZE][SIZE]) {
    char prepared[500];
    char digraph[3];
    int i, j = 0;
    
    // Prepare the plaintext
    prepareText(plaintext, prepared);
    
    // Process the text in pairs
    for(i = 0; prepared[i] != '\0'; i += 2) {
        char first = prepared[i];
        char second;
        
        // If we're at the last character or next char is same as current
        if(prepared[i+1] == '\0' || prepared[i] == prepared[i+1]) {
            if(prepared[i+1] == '\0') {
                // Add 'X' at the end if odd length
                second = 'X';
                i--; // Adjust index to process correctly
            }
            else {
                // Insert 'X' between duplicate letters
                second = 'X';
            }
        }
        else {
            second = prepared[i+1];
        }
        
        // Encrypt the pair
        encryptPair(keyTable, first, second, digraph);
        ciphertext[j++] = digraph[0];
        ciphertext[j++] = digraph[1];
    }
    ciphertext[j] = '\0';
}

// Decrypt the ciphertext
void decrypt(char *ciphertext, char *plaintext, char keyTable[SIZE][SIZE]) {
    char prepared[500];
    char digraph[3];
    int i, j = 0;
    
    // Prepare the ciphertext
    prepareText(ciphertext, prepared);
    
    // Process the text in pairs
    for(i = 0; prepared[i] != '\0'; i += 2) {
        if(prepared[i+1] == '\0')
            break;
            
        decryptPair(keyTable, prepared[i], prepared[i+1], digraph);
        plaintext[j++] = digraph[0];
        plaintext[j++] = digraph[1];
    }
    plaintext[j] = '\0';
}
```




## Output:
```
=== PLAYFAIR CIPHER IMPLEMENTATION ===

Enter the key: monarchy

Generated Key Table (5x5):
M O N A R
C H Y B D
E F G I K
L P Q S T
U V W X Z

1. Encrypt
2. Decrypt
Enter your choice (1 or 2): 1

Enter the plaintext: hello world

Encrypted text: FNEVLNQIPI
```
