#include <iostream>
#include <vector>
#include <string>
#include <ctime>
#include <algorithm>
using namespace std;

// 用户类
class User {
public:
    string username;
    string password;
    string role; // "admin", "teacher", "student"

    User(string name, string pwd, string r) : username(name), password(pwd), role(r) {}

    bool authenticate(const string& inputUsername, const string& inputPassword) {
        return username == inputUsername && password == inputPassword;
    }

    string getRole() const {
        return role;
    }
};

// 图书类
class Book {
public:
    string title;
    string author;
    string isbn;
    bool isElectronic;
    bool isBorrowed;
    User* borrower;
    string borrowDate;
    string returnDate;

    Book(string t, string a, string i, bool elec)
        : title(t), author(a), isbn(i), isElectronic(elec), isBorrowed(false), borrower(nullptr), borrowDate(""), returnDate("") {}

    void borrow(User* user) {
        if (!isBorrowed) {
            isBorrowed = true;
            borrower = user;
            borrowDate = getCurrentDateTime();
        }
    }

    void returnBook() {
        if (isBorrowed) {
            isBorrowed = false;
            borrower = nullptr;
            returnDate = getCurrentDateTime();
        }
    }

    static string getCurrentDateTime() {
        time_t now = time(0);
        tm* localTime = localtime(&now);
        char buffer[80];
        strftime(buffer, 80, "%Y-%m-%d %X", localTime);
        return string(buffer);
        
    }
};


// 图书馆管理类
class Library  { 
private:
    vector<Book> books;
    vector<User> users;

public:
    Library(vector<Book> b, vector<User> u) : books(b), users(u) {}
    
    Book* findBookByISBN(const string& isbn) {
    for (auto& book : books) {
        if (book.isbn == isbn) {
            return &book;
        }
    }
    return nullptr; // 如果没有找到图书
}
    User* findUserByUsername(const string& username) {
        for (auto& user : users) {
            if (user.username == username) {
                return &user;
            }
        }
        return nullptr; // 如果没有找到用户
    }

 bool login(const string& username, const string& password) {
        for (auto& user : users) {
            if (user.authenticate(username, password)) {
                return true;
            }
        }
        return false;
    }
bool addBook(const string& title, const string& author, const string& isbn, bool isElectronic) {
        for (const auto& existingBook : books) {
            if (existingBook.isbn == isbn) {
                cout << "图书已存在，无法添加重复的图书。" << endl;
                return false;
            }
        }
        books.push_back(Book(title, author, isbn, isElectronic));
        cout << "图书 '" << title << "' 添加成功！" << endl;
        return true;
    }
    void searchBooks(const string& keyword, bool electronicOnly = false) {
        for (const auto& book : books) {
            if (book.title.find(keyword) != string::npos ||
                book.author.find(keyword) != string::npos ||
                book.isbn.find(keyword) != string::npos) {
                bool isRelevant = !electronicOnly || book.isElectronic;
                if (isRelevant) {
                    cout << "Title: " << book.title
                              << ", Author: " << book.author
                              << ", ISBN: " << book.isbn
                              << ", Electronic: " << (book.isElectronic ? "Yes" : "No")
                              << ", Borrowed: " << (book.isBorrowed ? "Yes" : "No") << endl;
                }
            }
        }
    }

bool borrowBook(const string& isbn, User* borrower) {
    for (auto& book : books) {
        if (book.isbn == isbn) {
            if (!book.isBorrowed) {
                if (borrower->getRole() == "student" && book.isElectronic) {
                    cout << "学生不能借阅电子书。" << endl;
                    return false;
                }
                book.borrow(borrower);
                cout << "借阅成功！图书 '" << book.title << "' 已借给 " << borrower->username << "。" << endl;
                return true;
            } else {
                cout << "这本书已经被借走了。" << endl;
                return false;
            }
        }
    }
    cout << "未找到图书。" << endl;
    return false;
}

    bool returnBook(const string& isbn) {
    for (auto& book : books) {
        if (book.isbn == isbn) {
            if (book.isBorrowed) {
                book.returnBook();
                cout << "还书成功！图书 '" << book.title << "' 已成功归还。" << endl;
                return true;
            } else {
                cout << "这本书没有被借走，无需归还。" << endl;
                return false;
            }
        }
    }
    cout << "未找到图书，无法归还。" << endl;
    return false;
}

    bool removeBook(const string& isbn, const string& adminUsername) {
        for (auto it = users.begin(); it != users.end(); ++it) {
            if (it->username == adminUsername && it->role == "admin") {
                auto bookIt = find_if(books.begin(), books.end(), [isbn](const Book& book) {
                    return book.isbn == isbn;
                });
                if (bookIt != books.end()) {
                    if (!bookIt->isBorrowed) {
                        books.erase(bookIt);
                        cout << "图书删除成功！" << endl;
                        return true;
                    } else {
                        cout << "无法删除已借出的图书。" << endl;
                        return false;
                    }
                }
                cout << "未找到图书。" << endl;
                return false;
            }
        }
        cout << "只有管理员可以删除图书。" << endl;
        return false;
    }

    void editBook(const string& isbn, const string& adminUsername) {
        // 编辑图书信息的界面和逻辑可以根据需求设计
        for (auto it = users.begin(); it != users.end(); ++it) {
            if (it->username == adminUsername && it->role == "admin") {
                auto& book = *find_if(books.begin(), books.end(), [isbn](const Book& book) {
                    return book.isbn == isbn;
                });
                if (&book != nullptr) {
                    // 假设我们只允许编辑书名
                    cout << "请输入新的书名: ";
                    string newTitle;
                    cin >> newTitle;
                    book.title = newTitle;
                    cout << "图书编辑成功！" << endl;
                } else {
                    cout << "未找到图书。" << endl;
                }
                return;
            }
        }
        cout << "只有管理员可以编辑图书。" << endl;
    }
};

//  main 函数的代码 
int main() {
    // 初始化用户和图书数据
    std::vector<User> users = {
        User("admin", "admin123", "admin"),
        User("teacher", "teacher123", "teacher"),
        User("student", "student123", "student")
    };

    vector<Book> books = {
        Book("C++ Primer", "Stanley B. Lippman", "123456789", false),
        Book("Effective Modern C++", "Scott Meyers", "987654321", false),
        Book("Clean Code", "Robert C. Martin", "113456789", true)
    };

    Library lib(books, users);

    string username, password, keyword, isbn, title, author;
    char choice;
    string currentUsername; // 用于记录当前登录用户的用户名
    User* user = nullptr; // 声明用户指针

    while (true) {
        cout << "输入用户名: ";
        cin >> username;
        cout << "输入密码: ";
        cin >> password;

        if (!lib.login(username, password)) {
            cout << "用户名或密码错误。访问被拒绝。" << endl;
            break;
        }

        currentUsername = username; // 登录成功的用户名
        user = lib.findUserByUsername(currentUsername); // 初始化用户指针
        if (user) {
            cout << "登录成功！" << endl;
        } else {
            cout << "用户未找到。" << endl;
            break;
        }

        string role = user->getRole(); // 获取当前用户的角色

        do {
            cout << "\n欢迎使用图书馆管理系统\n";
            cout << "1. 查询图书\n";
            if (role != "admin") {
                cout << "2. 借书\n";
                cout << "3. 还书\n";
            }
            if (role == "admin") {
                cout << "4. 添加图书\n";
                cout << "5. 删除图书\n";
                cout << "6. 编辑图书\n";
            }
            cout << "0. 退出系统\n";
            cout << "请选择操作：";
            cin >> choice;

            switch (choice) {
                case '1':
                    cout << "输入查询关键字: ";
                    cin >> keyword;
                    lib.searchBooks(keyword);
                    break;

                case '2':
                    cout << "输入要借阅图书的ISBN号: ";
                    cin >> isbn;
                    if (user) {
                        lib.borrowBook(isbn, user);
                    }
                    break;
                case '3': {
                    if (role == "admin") {
                        cout << "管理员没有借书还书权限。" << endl;
                        break;
                    }
                    string action = (choice == '2') ? "输入要借阅图书的ISBN号: " : "输入要归还图书的ISBN号: ";
                    cout << action;
                    cin >> isbn;
                    if (choice == '2') {
                        lib.borrowBook(isbn, user);
                    } else {
                        lib.returnBook(isbn);
                    }
                    break;
                }

                 case '4':
                    if (user && user->role == "admin") {
                        char isElectronicInput;
                        cout << "输入图书标题: ";
                        cin >> title;
                        cout << "输入作者名: ";
                        cin >> author;
                        cout << "输入ISBN号: ";
                        cin >> isbn;
                        cout << "是否为电子书(y/n)? ";
                        cin >> isElectronicInput;
                        bool isElectronic = isElectronicInput == 'y' || isElectronicInput == 'Y';
                        lib.addBook(title, author, isbn, isElectronic);
                    } else {
                        cout << "只有管理员可以添加图书。" << endl;
                    }
                    break;

                case '5':
                    if (user && user->role == "admin") {
                        cout << "输入要删除的图书的ISBN号: ";
                        cin >> isbn;
                        lib.removeBook(isbn, currentUsername);
                    } else {
                        cout << "只有管理员可以删除图书。" << endl;
                    }
                    break;

                case '6':
                    if (user && user->role == "admin") {
                        cout << "输入要编辑的图书的ISBN号: ";
                        cin >> isbn;
                        Book* book = lib.findBookByISBN(isbn);
                        if (book) {
                            cout << "请输入新的书名: ";
                            cin >> book->title;
                            cout << "请输入新的作者名: ";
                            cin >> book->author;
                            cout << "图书编辑成功！" << endl;
                        } else {
                            cout << "未找到图书。" << endl;
                        }
                    } else {
                        cout << "只有管理员可以编辑图书。" << endl;
                    }
                    break;

                case '0':
                    cout << "正在退出系统。" << endl;
                    break;

                default:
                    cout << "无效的选择，请重新输入。" << endl;
                    break;
            }
        } while (choice != '0');

        cout << "感谢您使用图书馆系统。" << endl;
    }

    return 0;
}
