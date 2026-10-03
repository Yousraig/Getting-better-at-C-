#include <iostream>
#include <string>
#include <fstream>
#include <cstdio>      // remove()
#include <algorithm>   // swap()
using namespace std;


const string RESET = "\033[0m";
const string BOLD  = "\033[1m";
const string CYAN  = "\033[36m";
const string GREY  = "\033[90m";
const string RED   = "\033[91m";
const string CLEAR = "\033[2J\033[H";

// prints each line of a banner in the colour given for it
void printBanner(const string lines[], int count, const string colours[]) {
    for (int i = 0; i < count; ++i)
        cout << BOLD << colours[i % 6] << lines[i] << RESET << "\n";
}

void printTitle() {
    const string lines[5] = {
        " ____  _   _ ____   ___  _  ___   _ ",
        "/ ___|| | | |  _ \\ / _ \\| |/ / | | |",
        "\\___ \\| | | | | | | | | | ' /| | | |",
        " ___) | |_| | |_| | |_| | . \\| |_| |",
        "|____/ \\___/|____/ \\___/|_|\\_\\\\___/ "
    };
    // one solid pink for every line (256-colour palette). If your terminal ignores it, use "\033[95m".
    const string PINK = "\033[38;5;213m";
    const string colours[6] = { PINK, PINK, PINK, PINK, PINK, PINK };
    printBanner(lines, 5, colours);
}

void printVictory() {
    const string lines[5] = {
        "__     _____ ____ _____ ___  ______   __",
        "\\ \\   / /_ _/ ___|_   _/ _ \\|  _ \\ \\ / /",
        " \\ \\ / / | | |     | || | | | |_) \\ V / ",
        "  \\ V /  | | |___  | || |_| |  _ < | |  ",
        "   \\_/  |___\\____| |_| \\___/|_| \\_\\|_|  "
    };
    // one solid bright yellow for every line
    const string YELLOW = "\033[93m";
    const string colours[6] = { YELLOW, YELLOW, YELLOW, YELLOW, YELLOW, YELLOW };
    printBanner(lines, 5, colours);
    cout << BOLD << "  Board complete and correct. Well done!" << RESET << "\n";
}


// We need to check the legality of a move
bool isLegal(const int board[9][9], int row, int col, int value) {

    if (col < 0 || col >= 9 || row < 0 || row >= 9 || value < 0 || value > 9) {
        return false;
    }
    if (value == 0) {
        return true;
    }

    for (int x = 0; x < 9; ++x) {
        if (x != col && board[row][x] == value) return false;   // same row
        if (x != row && board[x][col] == value) return false;   // same column
    }

    int boxRow = (row / 3) * 3; // first row of the 3x3 square this cell is in (0, 3 or 6)
    int boxCol = (col / 3) * 3; // first column of that same square (0, 3 or 6)
    for (int x = 0; x < 3; ++x) {
        for (int y = 0; y < 3; ++y) {
            int r = boxRow + x; // actual row of the cell i'm looking at
            int c = boxCol + y; // actual column of the cell i'm looking at
            if ((r != row || c != col) && board[r][c] == value) return false;   // same number in my square
        }
    }
    return true;
}

// Makes sure that there are no illegal moves on the board
// checks the whole board rather than just one move
bool isConsistent(const int board[9][9]) {
    for (int row = 0; row < 9; ++row)
        for (int col = 0; col < 9; ++col)
            if (board[row][col] != 0 && !isLegal(board, row, col, board[row][col]))
                return false;
    return true;
}

// Victory check: board full AND no rule broken
bool isSolved(const int board[9][9]) {
    for (int row = 0; row < 9; ++row)
        for (int col = 0; col < 9; ++col)
            if (board[row][col] == 0)
                return false;
    return isConsistent(board);
}


// save and load the game based on previous inputs

bool save(string filename, int board[9][9], bool given[9][9]) {
    ofstream fout;

    // opens a file
    fout.open(filename);
    if (fout.fail()) {
        fout.close();
        return false;
    }

    for (int row = 0; row < 9; ++row) {
        for (int col = 0; col < 9; ++col) {
            fout << board[row][col] << " ";
        }
        fout << "\n";
    }
    for (int row = 0; row < 9; ++row) {
        for (int col = 0; col < 9; ++col) {
            fout << (given[row][col] ? 1 : 0) << " ";
        }
        fout << "\n";
    }

    fout.close();
    return true;
}

// Returns false and sets 'error' if the file is missing or not a valid puzzle
// The board is only changed if the whole file is valid

bool load(string filename, int board[9][9], bool given[9][9], string& error) {
    ifstream fin;
    fin.open(filename);

    if (fin.fail()) {
        error = "cannot open the file";
        return false;
    }

    string digits;
    char ch;

    while (fin.get(ch)) //fin.get(ch) reads the next single character
    {
        if (ch >= '0' && ch <= '9') digits += ch;
        else if (ch == '.') digits += '0';
        else if (ch != ' ' && ch != '\n' && ch != '\r' && ch != '\t') {
            error = "unexpected character in the file";
            fin.close();
            return false;
        }
    }
    fin.close(); // read the file, keep the digits, treat . as empty, ignore spacing, and reject any other character

    if (digits.size() != 81 && digits.size() != 162) {
        error = "the file must contain exactly 81 cells (9 rows of 9)";
        return false;
    }

    int newBoard[9][9];
    bool newGiven[9][9];
    for (int i = 0; i < 81; ++i) {
        newBoard[i / 9][i % 9] = digits[i] - '0';
    }
    for (int i = 0; i < 81; ++i) {
        if (digits.size() == 162) newGiven[i / 9][i % 9] = (digits[81 + i] == '1');
        else newGiven[i / 9][i % 9] = (newBoard[i / 9][i % 9] != 0);   // fresh puzzle: filled cells are the givens
    }

    if (!isConsistent(newBoard)) {
        error = "the puzzle breaks the rules (a number repeats in a row, column or box)";
        return false;
    }

    for (int row = 0; row < 9; ++row) {
        for (int col = 0; col < 9; ++col) {
            board[row][col] = newBoard[row][col];
            given[row][col] = newGiven[row][col];
        }
    }
    return true;
}

// Drawing the board

void draw(const int board[9][9], const bool given[9][9]) {
    cout << "\n    a b c   d e f   g h i\n";
    cout << "  +-------+-------+-------+\n";
    for (int row = 0; row < 9; ++row) {
        if (row == 3 || row == 6) cout << "  +-------+-------+-------+\n";
        cout << row + 1 << " |";
        for (int col = 0; col < 9; ++col) {
            if (board[row][col] == 0)  cout << " " << GREY << "." << RESET;                  // empty
            else if (given[row][col])  cout << " " << BOLD << board[row][col] << RESET;     // puzzle number
            else                       cout << " " << CYAN << board[row][col] << RESET;     // your number
            if (col % 3 == 2) cout << " |";
        }
        cout << "\n";
    }
    cout << "  +-------+-------+-------+\n\n";
}


// AUTOSOLVER
// finds the first empty cell, tries all values 1 to 9 and recurses. If none work, backtrack.

bool solve(int board[9][9]) {
    for (int row = 0; row < 9; ++row) {
        for (int col = 0; col < 9; ++col) {

            if (board[row][col] == 0) {
                for (int value = 1; value <= 9; ++value) {
                    if (isLegal(board, row, col, value)) {
                        board[row][col] = value;     // try it
                        if (solve(board)) {
                            return true;
                        }
                        board[row][col] = 0;         // it didn't work out so undo
                    }
                }
                return false;                        // nothing fits this cell
            }

        }
    }
    return true;                                     // no empty cell left and its therefore solved
}


// A built-in puzzle, so the game can start (or restart) without needing any file
void newGame(int board[9][9], bool given[9][9]) {
    int puzzle[9][9] = {
        {5, 3, 0,  0, 7, 0,  0, 0, 0},
        {6, 0, 0,  1, 9, 5,  0, 0, 0},
        {0, 9, 8,  0, 0, 0,  0, 6, 0},

        {8, 0, 0,  0, 6, 0,  0, 0, 3},
        {4, 0, 0,  8, 0, 3,  0, 0, 1},
        {7, 0, 0,  0, 2, 0,  0, 0, 6},

        {0, 6, 0,  0, 0, 0,  2, 8, 0},
        {0, 0, 0,  4, 1, 9,  0, 0, 5},
        {0, 0, 0,  0, 8, 0,  0, 7, 9}
    };
    for (int row = 0; row < 9; ++row) {
        for (int col = 0; col < 9; ++col) {
            board[row][col] = puzzle[row][col];
            given[row][col] = (puzzle[row][col] != 0);   // the starting numbers are locked
        }
    }
}


void alphabetize(string arr[], int size) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}


int main() {

    int numSaves = 0;
    string savedGames[1000];

    ifstream fin("directory.txt");
    if (!fin.fail()) {
        fin >> numSaves;
        if (numSaves < 0 || numSaves > 1000) numSaves = 0;
        for (int i = 0; i < numSaves; i++) {
            fin >> savedGames[i];
        }
        fin.close();
    }

    alphabetize(savedGames, numSaves);


    int board[9][9] = {};
    bool given[9][9] = {};
    string error;
    // continue the last game if there is an autosave, otherwise start the built-in puzzle
    if (!load("autosave.txt", board, given, error)) {
        newGame(board, given);
    }
    bool running = true;
    string command;
    string message;          // an error/info message shown under the board on the next screen
    bool clearNext = true;   // false after solve / listSaves so their output stays on screen

    while (running) {

        if (clearNext) cout << CLEAR;
        clearNext = true;

        printTitle();
        if (isSolved(board)) printVictory();
        draw(board, given);
        if (message != "") cout << message << "\n\n";
        message = "";

          cout << "To place a number type: set b4 7   (column letter, row number, value; 0 clears a cell)\n";
            cout << "Other commands:\n";
            cout << "  new                          Restart the game\n";
            cout << "  solve                        the computer solves it for you\n";
            cout << "  save (name the file)         save your game\n";
            cout << "  load (name of file)          load a puzzle or saved game\n";
            cout << "  listSaves                    show saved games\n";
            cout << "  delete (name of file)        delete a saved game\n";
            cout << "  exit                         quit (the game is autosaved)\n";
            cout << "Enter a command: ";
                    getline(cin, command);



        if (command.substr(0, 4) == "load")
        {
            string filename = "default.txt";
            if (command.length() > 4)
            {
                filename = command.substr(4);
                while (filename.length() > 0 && filename[0] == ' ') filename.erase(0, 1);
            }
            if (!load(filename, board, given, error))
                message = "Could not load '" + filename + "': " + error;
        }
        else if (command.substr(0, 4) == "save")
        {
            string filename = "default.txt";
            if (command.length() > 4)
            {
                filename = command.substr(4);
                while (filename.length() > 0 && filename[0] == ' ') filename.erase(0, 1);
            }

            if (!save(filename, board, given)) {
                message = "Could not save to '" + filename + "'.";
            }
            else if (filename != "default.txt")
            {
                // only add the name if it isn't already in the directory
                bool inDirectory = false;
                for (int i = 0; i < numSaves; ++i)
                    if (savedGames[i] == filename) inDirectory = true;

                if (!inDirectory && numSaves < 1000)
                {
                    savedGames[numSaves] = filename;
                    numSaves++;
                    alphabetize(savedGames, numSaves);   // keep the list in order
                }
            }
        }

        else if (command == "new") {
            newGame(board, given);     // back to the built-in puzzle
        }

        else if (command == "listSaves") {
            clearNext = false;
            cout << "Saved Games:" << endl;
            for (int i = 0; i < numSaves; ++i)
                cout << "\t" << savedGames[i] << endl;
        }

        else if (command.substr(0, 6) == "delete")
        {
            if (command.length() > 6)
            {
                string filename = command.substr(6);
                while (filename.length() > 0 && filename[0] == ' ')
                    filename.erase(0, 1);
                for (int i = 0; i < numSaves; ++i)
                {
                    if (savedGames[i] == filename)
                    {
                        for (int j = i + 1; j < numSaves; ++j)
                            savedGames[j - 1] = savedGames[j];
                        --numSaves;
                        break;   // stop once found
                    }
                }
                remove(filename.c_str());
            }
        }

        else if (command == "solve") {
            // work on a COPY so the player's board is never touched
            int solution[9][9];
            for (int row = 0; row < 9; ++row)
                for (int col = 0; col < 9; ++col)
                    solution[row][col] = board[row][col];

            if (solve(solution)) {
                clearNext = false;
                cout << "\nSolved by the algorithm. Your own board is unchanged:\n";
                draw(solution, given);
            }
            else {
                message = "This grid has no solution (if you have added your own numbers, one of them may be wrong).";
            }
        }

        else if (command == "exit") {
            save("autosave.txt", board, given);

            // write the directory so it is still there next run
            ofstream dout("directory.txt");
            dout << numSaves << "\n";
            for (int i = 0; i < numSaves; ++i) dout << savedGames[i] << "\n";
            dout.close();

            running = false;
        }

        else if (command.size() == 8 && command.substr(0, 4) == "set ") {

            int col = command[4] - 'a';
            int row = command[5] - '1';
            int value = command[7] - '0';

            if (col < 0 || col > 8 || row < 0 || row > 8 || value < 0 || value > 9) {
                message = "Out of range. Use columns a-i, rows 1-9, values 0-9 (0 clears a cell).";
            }

            else if (given[row][col]) {
                message = "That cell is part of the puzzle and can't be changed.";
            }

            else if (isLegal(board, row, col, value)) {
                board[row][col] = value;
            }

            else {
                message = RED + "This move is wrong, think again!" + RESET;
            }
        }

        else {
            message = "Unknown command. Use: set b4 7, new, solve, save, load, listSaves, delete or exit";
        }
    }
}
