```c
#include <stdio.h>
#include <stdlib.h>

/* Renamed to snake_case to keep function names consistent. */
static void swap_elements(int *a, int *b)
{
    int temp = *a;
    *a = *b;
    *b = temp;
}

/* Kept the original partition logic and made variable names clearer. */
static int partition_array(int array[], int low, int high)
{
    int pivot = array[high];
    int boundary = low - 1;

    for (int index = low; index < high; index++) {
        if (array[index] < pivot) {
            boundary++;
            swap_elements(&array[boundary], &array[index]);
        }
    }

    swap_elements(&array[boundary + 1], &array[high]);
    return boundary + 1;
}

/* Renamed from QuickSort to match the snake_case naming style. */
static void quick_sort(int array[], int low, int high)
{
    if (low < high) {
        int pivot_index = partition_array(array, low, high);
        quick_sort(array, low, pivot_index - 1);
        quick_sort(array, pivot_index + 1, high);
    }
}

/* Renamed from printArray; const indicates the function does not modify the array. */
static void print_array(const int array[], size_t size)
{
    for (size_t index = 0; index < size; index++) {
        /* Avoids leaving a trailing space after the last value. */
        printf("%d%s", array[index], index + 1 == size ? "\n" : " ");
    }
}

int main(void)
{
    /* Use a local array instead of malloc, so allocation failure and free are unnecessary. */
    int values[] = {10, 7, 8, 9, 1, 5, 2, 3, 4, 6};
    size_t size = sizeof(values) / sizeof(values[0]);

    printf("Original array:\n");
    print_array(values, size);

    /* Sort from the first element through the last. */
    quick_sort(values, 0, (int)size - 1);

    printf("Sorted array:\n");
    print_array(values, size);

    /* Return the standard success status from stdlib.h. */
    return EXIT_SUCCESS;
}
```
