#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <algorithm>
#include <windows.h>

using namespace std;

struct Student {
    string name;
    vector<int> grades;
};

// Функция для добавления ученика
void addStudent(vector<Student>& students) {
    string name;
    cout << "Введите фамилию и инициалы ученика: ";
    cin.ignore();
    getline(cin, name);

    students.push_back({ name, {} });
    cout << "Ученик добавлен.\n";
}

//+ оценка
void addGrades(vector<Student>& students) {
    string name;
    cout << "Введите фамилию и инициалы ученика, для которого нужно добавить оценки: ";
    cin.ignore();
    getline(cin, name);

    auto it = find_if(students.begin(), students.end(), [&](const Student& s) {
        return s.name == name;
        });

    if (it != students.end()) {
        vector<int> grades;
        int numOfGrades;

        cout << "Введите количество оценок: ";
        cin >> numOfGrades;

        for (int i = 0; i < numOfGrades; ++i) {
            int grade;
            cout << "Введите оценку №" << i + 1 << ": ";
            cin >> grade;

            // Проверка на допустимость оценки
            if (grade < 2 || grade > 5) {
                cout << "Ошибка: недопустимая оценка!\n";
                return;
            }

            grades.push_back(grade);
        }

        it->grades = grades;
        cout << "Оценки добавлены.\n";
    }
    else {
        cout << "Ошибка: ученик не найден!\n";
    }
}

//удаление ученика, зачем я пишу, если функция так и написана?
void removeStudent(vector<Student>& students) {
    string name;
    cout << "Введите фамилию и инициалы ученика, которого нужно удалить: ";
    cin.ignore();
    getline(cin, name);

    auto it = find_if(students.begin(), students.end(), [&](const Student& s) {
        return s.name == name;
        });

    if (it != students.end()) {
        students.erase(it);
        cout << "Ученик удален.\n";
    }
    else {
        cout << "Ошибка: ученик не найден!\n";
    }
}

// удаление оценок ученика
void removeGrades(vector<Student>& students) {
    string name;
    cout << "Введите фамилию и инициалы ученика, у которого нужно удалить оценки: ";
    cin.ignore();
    getline(cin, name);

    auto it = find_if(students.begin(), students.end(), [&](const Student& s) {
        return s.name == name;
        });

    if (it != students.end()) {
        it->grades.clear();
        cout << "Оценки удалены.\n";
    }
    else {
        cout << "Ошибка: ученик не найден!\n";
    }
}

//среднии оценки ученика
float calculateAverageGrade(const vector<int>& grades) {
    int sum = 0;
    for (int grade : grades) {
        sum += grade;
    }
    return static_cast<float>(sum) / grades.size();
}

//итоговой оценки ученика
void calculateTotalGrade(const vector<Student>& students) {
    string name;
    cout << "Введите фамилию и инициалы ученика, для которого нужно посчитать итоговую оценку: ";
    cin.ignore();
    getline(cin, name);

    auto it = find_if(students.begin(), students.end(), [&](const Student& s) {
        return s.name == name;
        });

    if (it != students.end()) {
        if (it->grades.empty()) {
            cout << "Ошибка: у ученика нет оценок!\n";
        }
        else {
            float average = calculateAverageGrade(it->grades);
            cout << "Итоговая оценка ученика " << it->name << ": " << average << "\n";
        }
    }
    else {
        cout << "Ошибка: ученик не найден!\n";
    }
}

//сохранения данных в файл перед выходом из программы
void saveDataOnExit(const vector<Student>& students, const string& filename) {
    ofstream file(filename);
    if (file.is_open()) {
        for (const Student& student : students) {
            file << student.name << " ";
            for (int grade : student.grades) {
                file << grade << " ";
            }
            file << "\n";
        }
        file.close();
        cout << "Данные сохранены в файл перед выходом.\n";
    }
    else {
        cout << "Ошибка: не удалось открыть файл для сохранения данных.\n";
    }
}
//экспорта данных в форматы .csv, .xlsx, .html, .json
bool exportData(const vector<Student>& students, const string& filename, const string& format) {
    // ваш код экспорта данных
    cout << "Данные экспортированы в файл " << filename << " в формате " << format << ".\n";
    return true;
}
int main() {
    SetConsoleCP(1251);
    SetConsoleOutputCP(1251);
    vector<Student> students;
    string filename = "journal.txt";

    // Загрузка данных из файла
    ifstream file(filename);
    if (file.is_open()) {
        string line;
        while (getline(file, line)) {
            size_t spaceIndex = line.find(" ");
            string name = line.substr(0, spaceIndex);
            string gradesStr = line.substr(spaceIndex + 1);
            vector<int> grades;

            size_t gradeIndex = 0;
            while (gradeIndex < gradesStr.size()) {
                size_t nextGradeIndex = gradesStr.find(" ", gradeIndex);
                string gradeStr = gradesStr.substr(gradeIndex, nextGradeIndex - gradeIndex);
                int grade = stoi(gradeStr);

                // Проверка на допустимость оценки
                if (grade < 2 || grade > 5) {
                    cout << "В файле обнаружена ошибка: недопустимая оценка!\n";
                    file.close();
                    return 1;
                }

                grades.push_back(grade);
                gradeIndex = nextGradeIndex + 1;
            }

            students.push_back({ name, grades });
        }
        file.close();
        cout << "Данные загружены из файла " << filename << ".\n";
    }
    else {
        cout << "Файл " << filename << " не найден. Создан новый журнал.\n";
    }

    // Главный цикл программы
    while (true) {
        cout << "\nВыберите действие:\n";
        cout << "1. Добавить ученика\n";
        cout << "2. Добавить оценки для ученика\n";
        cout << "3. Удалить ученика\n";
        cout << "4. Удалить оценки ученика\n";
        cout << "5. Посчитать итоговую оценку ученика\n";
        cout << "6. Выйти из программы\n";
        cout << "7. экспорт\n";
        cout << "Ваш выбор: ";

        int choice;
        cin >> choice;

        switch (choice) {
        case 1:
            addStudent(students);
            break;
        case 2:
            addGrades(students);
            break;
        case 3:
            removeStudent(students);
            break;
        case 4:
            removeGrades(students);
            break;
        case 5:
            calculateTotalGrade(students);
            break;
        case 6:
            saveDataOnExit(students, filename);
            return 0;
        case 7:
        {
            cout << "Выберите формат экспорта данных:\n";
            cout << "1. CSV\n";
            cout << "2. XLSX\n";
            cout << "3. HTML\n";
            cout << "4. JSON\n";

            int exportChoice;
            cin >> exportChoice;

            string exportFormat;
            string exportFilename;

            switch (exportChoice) {
            case 1:
                exportFormat = "CSV";
                cout << "Введите имя файла для экспорта: ";
                cin >> exportFilename;
                exportData(students, exportFilename, exportFormat);
                break;
            case 2:
                exportFormat = "XLSX";
                cout << "Введите имя файла для экспорта: ";
                cin >> exportFilename;
                exportData(students, exportFilename, exportFormat);
                break;
            case 3:
                exportFormat = "HTML";
                cout << "Введите имя файла для экспорта: ";
                cin >> exportFilename;
                exportData(students, exportFilename, exportFormat);
                break;
            case 4:
                exportFormat = "JSON";
                cout << "Введите имя файла для экспорта: ";
                cin >> exportFilename;
                exportData(students, exportFilename, exportFormat);
                break;
            default:
                cout << "Неверный выбор формата экспорта.\n";
                break;
            }

            break;
        }
        default:
            cout << "Неверный выбор. Попробуйте еще раз.\n";
            break;
        }
    }

    return 0;
}
