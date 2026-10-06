#include <stdio.h>
#include <string.h>
#include <math.h>

// Automatically detects Windows or Linux/macOS for the frame delay
#ifdef _WIN32
#include <windows.h>
#define sleep_ms(x) Sleep(x)
#else
#include <unistd.h>
#define sleep_ms(x) usleep(x * 1000)
#endif

int main() {
    float A = 0, B = 0;
    float i, j;
    int k;
    
    // FIXED: Turned into arrays of size 1760 to hold the screen pixels
    float z[1760];
    char b[1760];
    
    // Clear the terminal screen entirely before beginning
    printf("\x1b[2J");
    
    for (;;) {
        // Reset the text buffer with spaces and depth buffer with 0
        memset(b, 32, 1760);
        memset(z, 0, 7040); // 1760 elements * 4 bytes per float = 7040 bytes
        
        // theta (j) and phi (i) loops to calculate donut geometry
        for (j = 0; j < 6.28; j += 0.07) {
            for (i = 0; i < 6.28; i += 0.02) {
                float c = sin(i);
                float d = cos(j);
                float e = sin(A);
                float f = sin(j);
                float g = cos(A);
                float h = d + 2;
                float D = 1 / (c * h * e + f * g + 5);
                float l = cos(i);
                float m = cos(B);
                float n = sin(B);
                float t = c * h * g - f * e;
                
                // Calculate 2D screen projection coordinates (x, y)
                int x = (int)(40 + 30 * D * (l * h * m - t * n));
                int y = (int)(12 + 15 * D * (l * h * n + t * m));
                int o = x + 80 * y;
                
                // Calculate surface luminance (lighting index)
                int N = (int)(8 * ((f * e - c * d * g) * m - c * d * e - f * g - l * d * n));
                
                // If coordinates fit inside screen bounds and are closer than previous depth
                if (22 > y && y > 0 && x > 0 && 80 > x && D > z[o]) {
                    z[o] = D;
                    // Match lighting index to ASCII characters for beautiful shading
                    b[o] = ".,-~:;=!*#$@"[N > 0 ? N : 0];
                }
            }
        }
        
        // Return cursor to the top-left home position to prevent flickering
        printf("\x1b[H");
        
        // Set text color to Glowing Matrix Green
        printf("\x1b[32m"); 
        
        // Print the frame buffer onto the console screen
        for (k = 0; k < 1761; k++) {
            putchar(k % 80 ? b[k] : 10);
        }
        
        // Increment rotation angles for the next frame
        A += 0.04;
        B += 0.02;
        
        // Short delay for smooth movement (30 milliseconds)
        sleep_ms(30);
    }
    return 0;
}
