/*
 * ============================================================
 *  PERSONAL EXPENSE TRACKER USING C
 *  Software Development Fundamentals Lab - 1  (PBL)
 *
 *  Group Members:
 *    Veer Chopra     (2601020108)
 *    Shaurya Agrawal (2601020117)
 *    Aradhy Agarwal  (2601020102)
 *    Naman Saini     (2601020107)
 *  Batch: A4
 *
 *  Concepts used: Structures, Arrays, Pointers, Functions,
 *                 File Handling, if / else if, Loops
 *
 *  Compile : gcc expense_tracker.c -o expense_tracker
 * ============================================================
 */

#include <stdio.h>

#define MAX 100                  /* maximum number of expenses */
#define FILE_NAME "expenses.txt"

/* ---------- STRUCTURE ---------- */
struct Expense
{
    int   id;
    int   day;
    int   month;
    int   year;
    int   category;              /* 1 to 5 (see getCategoryName) */
    char  description[50];
    float amount;
};

/* ====================== HELPER FUNCTIONS ====================== */

/* removes leftover characters (like Enter) from the keyboard buffer */
void clearBuffer()
{
    int ch;
    ch = getchar();
    while (ch != '\n' && ch != EOF)
    {
        ch = getchar();
    }
}

/* reads an integer between min and max; asks again if wrong.
   Input like "12abc" is also rejected (nothing may follow the number). */
int readInt(char prompt[], int min, int max)
{
    int value;
    int ch;
    int ok = 0;

    while (ok == 0)
    {
        printf("%s", prompt);
        if (scanf("%d", &value) == 1)
        {
            ch = getchar();              /* character right after the number */
            if (ch == '\n' && value >= min && value <= max)
            {
                ok = 1;
            }
            else
            {
                if (ch != '\n')
                    clearBuffer();       /* throw away the extra letters */
                printf("  Invalid input! Enter only a number from %d to %d.\n", min, max);
            }
        }
        else
        {
            clearBuffer();
            printf("  Invalid input! Enter only a number from %d to %d.\n", min, max);
        }
    }
    return value;
}

/* reads an amount greater than 0 (letters are rejected) */
float readAmount(char prompt[])
{
    float value;
    int ch;
    int ok = 0;

    while (ok == 0)
    {
        printf("%s", prompt);
        if (scanf("%f", &value) == 1)
        {
            ch = getchar();
            if (ch == '\n' && value > 0)
            {
                ok = 1;
            }
            else
            {
                if (ch != '\n')
                    clearBuffer();
                printf("  Invalid amount! Enter only a number greater than 0.\n");
            }
        }
        else
        {
            clearBuffer();
            printf("  Invalid amount! Enter only a number greater than 0.\n");
        }
    }
    return value;
}

/* returns 1 if the date is valid, otherwise 0 */
int isValidDate(int d, int m, int y)
{
    int daysInMonth;

    if (m == 2)
    {
        if ((y % 4 == 0 && y % 100 != 0) || (y % 400 == 0))
            daysInMonth = 29;
        else
            daysInMonth = 28;
    }
    else if (m == 4 || m == 6 || m == 9 || m == 11)
    {
        daysInMonth = 30;
    }
    else
    {
        daysInMonth = 31;
    }

    if (d >= 1 && d <= daysInMonth)
        return 1;
    else
        return 0;
}

/* returns the name of a category number (uses if - else if) */
char *getCategoryName(int c)
{
    if (c == 1)
        return "Food";
    else if (c == 2)
        return "Travel";
    else if (c == 3)
        return "Shopping";
    else if (c == 4)
        return "Education";
    else
        return "Other";
}

void showCategories()
{
    printf("  1. Food\n  2. Travel\n  3. Shopping\n  4. Education\n  5. Other\n");
}

/* finds position of an ID in the array, returns -1 if not found */
int findIndex(struct Expense list[], int n, int id)
{
    int i;
    for (i = 0; i < n; i++)
    {
        if (list[i].id == id)
            return i;
    }
    return -1;
}

void printHeader()
{
    printf("\n%-5s %-12s %-12s %-25s %10s\n",
           "ID", "Date", "Category", "Description", "Amount");
    printf("-------------------------------------------------------------------\n");
}

/* prints one record - receives a POINTER to the record */
void printExpense(struct Expense *e)
{
    printf("%-5d %02d-%02d-%-6d %-12s %-25s %10.2f\n",
           e->id, e->day, e->month, e->year,
           getCategoryName(e->category), e->description, e->amount);
}

/* takes input for one record - fills the record through a POINTER */
void inputDetails(struct Expense *e)
{
    int valid = 0;

    while (valid == 0)
    {
        printf("\nEnter date:\n");
        e->day   = readInt("  Day   (1-31)    : ", 1, 31);
        e->month = readInt("  Month (1-12)    : ", 1, 12);
        e->year  = readInt("  Year  (YYYY)    : ", 1900, 2100);
        valid = isValidDate(e->day, e->month, e->year);
        if (valid == 0)
            printf("  This date does not exist! Enter again.\n");
    }

    printf("\nSelect category:\n");
    showCategories();
    e->category = readInt("Enter category number (1-5): ", 1, 5);

    printf("Enter description: ");
    scanf(" %49[^\n]", e->description);
    clearBuffer();

    e->amount = readAmount("Enter amount (Rs.): ");
}

/* ====================== FILE HANDLING ====================== */

/* saves all records into a text file */
void saveToFile(struct Expense list[], int n)
{
    FILE *fp;
    int i;

    fp = fopen(FILE_NAME, "w");
    if (fp == NULL)
    {
        printf("  Error: could not open file for saving!\n");
        return;
    }

    fprintf(fp, "%d\n", n);
    for (i = 0; i < n; i++)
    {
        fprintf(fp, "%d %d %d %d %d %f %s\n",
                list[i].id, list[i].day, list[i].month, list[i].year,
                list[i].category, list[i].amount, list[i].description);
    }
    fclose(fp);
}

/* loads records from the file, returns how many were loaded */
int loadFromFile(struct Expense list[])
{
    FILE *fp;
    int n = 0;
    int i;

    fp = fopen(FILE_NAME, "r");
    if (fp == NULL)
        return 0;                /* first run - file does not exist yet */

    if (fscanf(fp, "%d", &n) != 1 || n < 0 || n > MAX)
    {
        fclose(fp);
        return 0;
    }

    for (i = 0; i < n; i++)
    {
        fscanf(fp, "%d %d %d %d %d %f %49[^\n]",
               &list[i].id, &list[i].day, &list[i].month, &list[i].year,
               &list[i].category, &list[i].amount, list[i].description);
    }
    fclose(fp);
    return n;
}

/* ====================== MAIN FEATURES ====================== */

/* 1. ADD EXPENSE  (count is passed as a POINTER so it can be changed) */
void addExpense(struct Expense list[], int *count)
{
    int i;
    int maxId = 0;

    if (*count >= MAX)
    {
        printf("\n  Storage full! Delete some records first.\n");
        return;
    }

    /* new ID = biggest existing ID + 1 */
    for (i = 0; i < *count; i++)
    {
        if (list[i].id > maxId)
            maxId = list[i].id;
    }
    list[*count].id = maxId + 1;

    inputDetails(&list[*count]);

    (*count)++;
    saveToFile(list, *count);
    printf("\n  Expense added successfully! (ID = %d)\n", maxId + 1);
}

/* 2. VIEW ALL EXPENSES */
void viewExpenses(struct Expense list[], int n)
{
    int i;

    if (n == 0)
    {
        printf("\n  No expense records found.\n");
        return;
    }

    printHeader();
    for (i = 0; i < n; i++)
    {
        printExpense(&list[i]);
    }
    printf("\n  Total records: %d\n", n);
}

/* 3. SEARCH EXPENSE (by ID or by category) */
void searchExpense(struct Expense list[], int n)
{
    int choice, i, id, cat, pos;
    int found = 0;

    printf("\nSearch by:\n  1. Expense ID\n  2. Category\n");
    choice = readInt("Enter choice: ", 1, 2);

    if (choice == 1)
    {
        id = readInt("Enter expense ID: ", 1, 100000);
        pos = findIndex(list, n, id);
        if (pos != -1)
        {
            printHeader();
            printExpense(&list[pos]);
            found = 1;
        }
    }
    else if (choice == 2)
    {
        showCategories();
        cat = readInt("Enter category number (1-5): ", 1, 5);
        for (i = 0; i < n; i++)
        {
            if (list[i].category == cat)
            {
                if (found == 0)
                    printHeader();
                printExpense(&list[i]);
                found = 1;
            }
        }
    }

    if (found == 0)
        printf("\n  No matching record found.\n");
}

/* 4. UPDATE EXPENSE */
void updateExpense(struct Expense list[], int n)
{
    int id, pos;

    if (n == 0)
    {
        printf("\n  No records to update.\n");
        return;
    }

    id = readInt("\nEnter ID of the expense to update: ", 1, 100000);
    pos = findIndex(list, n, id);

    if (pos == -1)
    {
        printf("  Record with ID %d not found.\n", id);
        return;
    }

    printf("\n  Current record:");
    printHeader();
    printExpense(&list[pos]);

    printf("\n  Enter the new details:");
    inputDetails(&list[pos]);       /* ID stays the same */

    saveToFile(list, n);
    printf("\n  Expense updated successfully!\n");
}

/* 5. DELETE EXPENSE */
void deleteExpense(struct Expense list[], int *count)
{
    int id, pos, i;
    char confirm;

    if (*count == 0)
    {
        printf("\n  No records to delete.\n");
        return;
    }

    id = readInt("\nEnter ID of the expense to delete: ", 1, 100000);
    pos = findIndex(list, *count, id);

    if (pos == -1)
    {
        printf("  Record with ID %d not found.\n", id);
        return;
    }

    printf("Are you sure you want to delete it? (y/n): ");
    scanf(" %c", &confirm);
    clearBuffer();

    if (confirm == 'y' || confirm == 'Y')
    {
        /* shift all later records one place to the left */
        for (i = pos; i < *count - 1; i++)
        {
            list[i] = list[i + 1];
        }
        (*count)--;
        saveToFile(list, *count);
        printf("  Expense deleted successfully!\n");
    }
    else
    {
        printf("  Deletion cancelled.\n");
    }
}

/* 6. TOTAL EXPENSE */
void totalExpense(struct Expense list[], int n)
{
    int i;
    float total = 0;

    for (i = 0; i < n; i++)
    {
        total = total + list[i].amount;
    }

    printf("\n  Total number of expenses : %d", n);
    printf("\n  Total amount spent       : Rs. %.2f\n", total);
}

/* 7. CATEGORY-WISE SUMMARY */
void categorySummary(struct Expense list[], int n)
{
    int i, c;
    float sum[6] = {0, 0, 0, 0, 0, 0};   /* index 1 to 5 used */
    int   num[6] = {0, 0, 0, 0, 0, 0};
    float grand = 0;

    if (n == 0)
    {
        printf("\n  No expense records found.\n");
        return;
    }

    for (i = 0; i < n; i++)
    {
        sum[list[i].category] = sum[list[i].category] + list[i].amount;
        num[list[i].category]++;
        grand = grand + list[i].amount;
    }

    printf("\n%-12s %8s %12s %10s\n", "Category", "Count", "Amount", "Share");
    printf("------------------------------------------------\n");
    for (c = 1; c <= 5; c++)
    {
        printf("%-12s %8d %12.2f %9.1f%%\n",
               getCategoryName(c), num[c], sum[c], (sum[c] / grand) * 100);
    }
    printf("------------------------------------------------\n");
    printf("%-12s %8d %12.2f\n", "TOTAL", n, grand);
}

/* 8. RESET - deletes ALL expenses permanently */
void resetAll(struct Expense list[], int *count)
{
    char confirm;

    if (*count == 0)
    {
        printf("\n  There is no data to reset.\n");
        return;
    }

    printf("\n  WARNING: This will permanently delete all %d record(s)!\n", *count);
    printf("  Are you sure? (y/n): ");
    scanf(" %c", &confirm);
    clearBuffer();

    if (confirm == 'y' || confirm == 'Y')
    {
        *count = 0;                      /* no records remain in memory */
        saveToFile(list, *count);        /* file now stores 0 records   */
        printf("  All expenses have been deleted. Tracker is now empty.\n");
    }
    else
    {
        printf("  Reset cancelled. Your data is safe.\n");
    }
}

/* ====================== MAIN ====================== */

void showMenu()
{
    printf("\n==========================================\n");
    printf("      PERSONAL EXPENSE TRACKER\n");
    printf("==========================================\n");
    printf("  1. Add Expense\n");
    printf("  2. View All Expenses\n");
    printf("  3. Search Expense\n");
    printf("  4. Update Expense\n");
    printf("  5. Delete Expense\n");
    printf("  6. Total Expense\n");
    printf("  7. Category-wise Summary\n");
    printf("  8. Reset (Delete All Data)\n");
    printf("  9. Exit\n");
    printf("------------------------------------------\n");
}

int main()
{
    struct Expense list[MAX];     /* ARRAY of structures */
    int count;
    int choice = 0;

    count = loadFromFile(list);
    printf("\n  Welcome! %d record(s) loaded from file.\n", count);

    while (choice != 9)
    {
        showMenu();
        choice = readInt("Enter your choice (1-9): ", 1, 9);

        if (choice == 1)
            addExpense(list, &count);
        else if (choice == 2)
            viewExpenses(list, count);
        else if (choice == 3)
            searchExpense(list, count);
        else if (choice == 4)
            updateExpense(list, count);
        else if (choice == 5)
            deleteExpense(list, &count);
        else if (choice == 6)
            totalExpense(list, count);
        else if (choice == 7)
            categorySummary(list, count);
        else if (choice == 8)
            resetAll(list, &count);
        else
            printf("\n  Data saved. Thank you for using Expense Tracker!\n");
    }

    saveToFile(list, count);
    return 0;
}

