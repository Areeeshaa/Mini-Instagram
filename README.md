```cpp
#include <iostream>
#include <string>
#include <ctime>
#include<iomanip>
#include<windows.h>
using namespace std;

// Utility function to get the current timestamp
string getCurrentTime() {
    time_t now = time(0);
    char buffer[26]; // Ensure enough space for the date and time string
    ctime_s(buffer, sizeof(buffer), &now); // Safer version of ctime
    return string(buffer); // Return the string without the newline
}

// Linked list node for posts
struct PostNode {
    string content;
    string timestamp;
    PostNode* next;

    PostNode(string txt) : content(txt), timestamp(getCurrentTime()), next(nullptr) {}
};

// Stack for posts
class PostStack {
private:
    PostNode* top;

public:
    PostStack() : top(nullptr) {}

    void push(string content) {
        PostNode* newNode = new PostNode(content);
        newNode->next = top;
        top = newNode;
    }

    bool isEmpty() {
        if (top == nullptr)
        {
            return true;
        }
        return false;
    }

    void display() {
        if (!top) {
            cout << "No posts available.\n";
            return;
        }
        PostNode* current = top;
        while (current) {
            cout << "[" << current->timestamp << "] " << current->content << "\n";
            current = current->next;
        }
    }
};
struct FollowRequestNode {
    string requester;
    FollowRequestNode* next;

    FollowRequestNode(string req) : requester(req), next(nullptr) {}
};



// Stack for messages
struct MessageNode {
    string sender;
    string content;
    string timestamp;
    MessageNode* next;

    MessageNode(string sndr, string txt) : sender(sndr), content(txt), timestamp(getCurrentTime()), next(nullptr) {}
};


class MessageStack {
private:
    MessageNode* top;

public:
    MessageStack() : top(nullptr) {}

    void push(string sender, string content) {
        MessageNode* newNode = new MessageNode(sender, content);
        newNode->next = top;
        top = newNode;
    }

    void display(const string& currentUser, const string& withUser) {
        if (!top) { // Check if the stack is empty
            cout << "No messages available.\n";
            return;
        }

        MessageNode* current = top; // Start from the top of the stack
        bool foundMessages = false; // Flag to track if messages were found
        const int consoleWidth = 79; // Adjust for alignment

        while (current) {
            // Display messages where the sender is the current user or the specified user
            if (current->sender == currentUser || current->sender == withUser) {
                foundMessages = true; // Mark that at least one message was found
                if (current->sender == currentUser) {
                    // Message sent by the current user (align to the right)
                    cout << "[" << current->timestamp << "] You: " << current->content << "\n";
                }
                else if (current->sender == withUser) {
                    // Message received from the specified user (align to the left)
                    cout << "[" << current->timestamp << "] " << current->sender << ": " << current->content << "\n";
                }
            }
            current = current->next; // Move to the next message
        }

        if (!foundMessages) { // If no messages were found
            cout << "No messages with " << withUser << ".\n";
        }
    }




};

class FollowRequestQueue {
private:
    FollowRequestNode* front;
    FollowRequestNode* rear;

public:
    FollowRequestQueue() : front(nullptr), rear(nullptr) {}

    void enqueue(string requester) {
        FollowRequestNode* newNode = new FollowRequestNode(requester);
        if (!rear) {
            front = rear = newNode;
        }
        else {
            rear->next = newNode;
            rear = newNode;
        }
    }

    void dequeue() {
        if (!front) {
            cout << "Queue is empty.\n";
            return;
        }
        FollowRequestNode* temp = front;
        front = front->next;
        delete temp;
        if (!front) rear = nullptr;
    }

    void display() {
        if (!front) {
            cout << "Queue is empty.\n";
            return;
        }
        FollowRequestNode* current = front;
        while (current) {
            cout << "- " << current->requester << endl;
            current = current->next;
        }
    }
};

// User node for the graph
struct User {
    string name;
    string password;
    string city;
    PostStack posts;
    MessageStack messages;
    FollowRequestQueue followRequests;
    FollowRequestQueue notifications;
    User* next; // Linked list of all users
    User* followers; // Adjacency list for followers
    User* following; // Adjacency list for following

    User(string nm, string pwd, string ct) : name(nm), password(pwd), city(ct), next(nullptr), followers(nullptr), following(nullptr) {}

};




// BST Node for storing users
struct BSTNode {
    string username;
    User* userPtr;  // Pointer to the corresponding user in the graph
    BSTNode* left;
    BSTNode* right;

    BSTNode(string name, User* user) : username(name), userPtr(user), left(nullptr), right(nullptr) {}
};

// BST class for storing users in sorted order
class UserBST {
private:
    BSTNode* root;

    // Helper function to insert a new node into the BST
    BSTNode* insert(BSTNode* node, string name, User* user) {
        if (node == nullptr) {
            return new BSTNode(name, user);
        }
        if (name < node->username) {
            node->left = insert(node->left, name, user);
        }
        else if (name > node->username) {
            node->right = insert(node->right, name, user);
        }
        return node;
    }

    // Helper function to search for a user by username
    User* search(BSTNode* node, string name) {
        if (node == nullptr) return nullptr;

        if (name == node->username) {
            return node->userPtr;
        }
        else if (name < node->username) {
            return search(node->left, name);
        }
        else {
            return search(node->right, name);
        }
    }

public:
    UserBST() : root(nullptr) {}

    // Public method to insert a new user into the BST
    void insert(string name, User* user) {
        root = insert(root, name, user);
    }

    // Public method to search for a user by username
    User* search(string name) {
        return search(root, name);
    }

    // In-order traversal to display all users in sorted order
    void displayInOrder(BSTNode* node) {
        if (node != nullptr) {
            displayInOrder(node->left);
            cout << node->username << "\n";
            displayInOrder(node->right);
        }
    }

    void displayUsers() {
        displayInOrder(root);
    }
};




// Queue for friend requests or notifications
struct QueueNode {
    string content;
    QueueNode* next;

    QueueNode(string txt) : content(txt), next(nullptr) {}
};

class RequestQueue {
private:
    QueueNode* front;
    QueueNode* rear;

public:
    RequestQueue() : front(nullptr), rear(nullptr) {}

    void enqueue(string content) {
        QueueNode* newNode = new QueueNode(content);
        if (!rear) {
            front = rear = newNode;
        }
        else {
            rear->next = newNode;
            rear = newNode;
        }
    }

    void dequeue() {
        if (!front) {
            cout << "Queue is empty.\n";
            return;
        }
        QueueNode* temp = front;
        front = front->next;
        delete temp;
        if (!front) rear = nullptr;
    }

    void display() {
        if (!front) {
            cout << "Queue is empty.\n";
            return;
        }
        QueueNode* current = front;
        while (current) {
            cout << "- " << current->content << "\n";
            current = current->next;
        }
    }
};
struct SearchHistoryNode {
    string username;
    SearchHistoryNode* next;

    SearchHistoryNode(string user) : username(user), next(nullptr) {}
};

// Class to manage search history
class SearchHistory {
private:
    SearchHistoryNode* head;

public:
    SearchHistory() : head(nullptr) {}

    // Add a search to the history
    void addSearch(string username) {
        // Create a new node
        SearchHistoryNode* newNode = new SearchHistoryNode(username);
        newNode->next = head;
        head = newNode;
    }

    // Display the search history
    void displayHistory() {
        if (!head) {
            cout << "No search history available.\n";
            return;
        }
        cout << "Search History:\n";
        SearchHistoryNode* current = head;
        while (current) {
            cout << "- " << current->username << "\n";
            current = current->next;
        }
    }
};


const int TABLE_SIZE = 101;


// Queue for follow requests or notifications
// Graph for user network
class UserGraph {
private:
    User* head;
    User* currentUser;  // Pointer to the logged-in user
    UserBST bst;
    SearchHistory searchHistory;  // Instance to store search history
    User* table[TABLE_SIZE];

public:
    UserGraph() : head(nullptr), currentUser(nullptr) {
        for (int i = 0; i < TABLE_SIZE; ++i) {
            table[i] = nullptr;
        }
    }

    User* searchUser(string name) {
        int index = hashFunction(name);
        User* recipient = table[index];
        if (currentUser) {
            searchHistory.addSearch(name);  // Log the search
        }

        return recipient;
    }

    void logout() {
        if (!currentUser) {
            cout << "Error: No user is currently logged in.\n";
            return;
        }

        cout << "Goodbye, " << currentUser->name << "!\n";
        currentUser = nullptr;
    }

    User* getCurrentUser() {
        return currentUser;
    }

    // Method to display the search history
    void displaySearchHistory() {
        if (!currentUser) {
            cout << "Error: Please login to view your search history.\n";
            return;
        }
        searchHistory.displayHistory();  // Call the display function of the SearchHistory class
    }



    void sendFollowRequest(string to) {
        if (!currentUser) {
            cout << "Error: Please login to send follow requests.\n";
            return;
        }
        User* recipient = searchUser(to);

        if (!recipient) {
            cout << "Invalid recipient.\n";
            return;
        }

        recipient->followRequests.enqueue(currentUser->name + " wants to follow you!");
        recipient->notifications.enqueue(currentUser->name + " sent you a follow request");
        cout << "Follow request sent from " << currentUser->name << " to " << to << ".\n";
    }

    void acceptFollowRequest(string requesterName) {
        if (!currentUser) {
            cout << "Error: Please login to accept follow requests.\n";
            return;
        }
        User* requester = searchUser(requesterName);
        if (!requester) {
            cout << "Invalid requester.\n";
            return;
        }

        // Add to following list of the current user
        User* newFollowingNode = new User(currentUser->name, "", "");
        newFollowingNode->next = requester->following;
        requester->following = newFollowingNode;

        // Add to followers list of the requester
        User* newFollowerNode = new User(requesterName, "", "");
        newFollowerNode->next = currentUser->followers;
        currentUser->followers = newFollowerNode;

        currentUser->notifications.enqueue(requesterName + " started following you.");
        cout << requesterName << " is now following " << currentUser->name << ".\n";
    }

    void displayFollowers(string name) {
        User* user = searchUser(name);
        if (!user) {
            cout << "User not found.\n";
            return;
        }
        cout << "Followers of " << name << ":\n";
        User* current = user->followers;
        while (current) {
            cout << "- " << current->name << "\n";
            current = current->next;
        }
    }

    void displayFollowing(string name) {
        User* user = searchUser(name);
        if (!user) {
            cout << "User not found.\n";
            return;
        }

        cout << "Following of " << name << ":\n";
        User* current = user->following;
        while (current) {
            cout << "- " << current->name << "\n";
            current = current->next;
        }
    }


    int hashFunction(const string& key) {
        int hash = 0;
        for (char ch : key) {
            hash = (hash * 31 + ch) % TABLE_SIZE;  // A simple hash function
        }
        return hash;
    }



    void insert(const string& username, const string& password, const string& ct) {
        int index = hashFunction(username);
        User* newNode = new User(username, password, ct);

        if (table[index] == nullptr) {
            table[index] = newNode;
        }
        else {
            // Collision handling: Add to the front of the linked list
            newNode->next = table[index];
            table[index] = newNode;
        }
    }

    bool verify(const string& username, const string& password) {
        int index = hashFunction(username);
        User* current = table[index];
        while (current != nullptr) {
            if (current->name == username && current->password == password) {
                currentUser = current;
                cout << "Welcome " << username << endl;
                return true;
            }
            current = current->next;
        }
        return false;
    }
    User* getUser(string name) {
        int index = hashFunction(name);
        User* recipient = table[index];

        return recipient;
    }
    void displayFollowerPosts()
    {
        if (currentUser == nullptr)
        {
            cout << "Please login to view follower posts.\n";
            return;
        }

        User* friendNode = currentUser->followers;
        bool hasPosts = false;

        cout << "Posts from your friends:\n";

        while (friendNode)
        {
            cout << "\nChecking friend: " << friendNode->name << "\n";

            User* friendUser = searchUser(friendNode->name);

            if (friendUser)
            {
                cout << "Found friend: " << friendUser->name << "\n";

                if (!friendUser->posts.isEmpty())
                {
                    cout << "Posts by " << friendUser->name << ":\n";
                    friendUser->posts.display();
                    hasPosts = true;
                }

                else
                {
                    cout << "No posts available for " << friendUser->name << ".\n";
                }
            }

            else
            {
                cout << "Error: Could not retrieve user " << friendNode->name << ".\n";
            }

            friendNode = friendNode->next;
        }

        if (!hasPosts)
        {
            cout << "None of your friends have posted yet.\n";
        }
    }
};

// Menu functionalities
void displayMenu() {
    cout << "\n------------------------------------------------Micro-Instagram Menu:--------------------------------------------------\n"
        << "1. Signup\n"
        << "2. Login\n"
        << "3. Exit\n";
}
void display2() {
    cout << "1. Logout\n"
        << "2. Send Follow Request\n"
        << "3. Accept Follow Request\n"
        << "4. Post Content\n"
        << "5. View Posts\n"
        << "6. Send Messages\n"
        << "7. View Messages\n"
        << "8. View Follow Requests\n"
        << "9. View Notifications\n"
        << "A. Display Followers\n"
        << "B. Display Following\n"
        << "C. Search Users\n"
        << "D. View Search History\n"
        << "E. Display Posts of Followers\n";
}

int main() {
    UserGraph userGraph;
    string name, password, city, friendName, content, recipient, message, followName;
    char choice;
    string log_name = "";
    int pass_len, index;
    bool flag = false;
    User* recipientUser;
    system("Color 0E");
    cout << "\n\n";
    for (int i = 0; i < 8; ++i) {
        cout << endl;
    }

    // Outer large box
    cout << setw(80) << "\n";
    cout << setw(80) << "*                                  *\n";
    cout << setw(80) << "*                                  *\n";
    cout << setw(80) << "*                                  *\n";

    // Mini box inside
    cout << setw(80) << "*       ***      *\n";
    cout << setw(80) << "*       *                   *      *\n";
    cout << setw(80) << "*       *    INSTAGRAM      *      *\n";
    cout << setw(80) << "*       *                   *      *\n";
    cout << setw(80) << "*       ***      *\n";

    cout << setw(80) << "*                                  *\n";
    cout << setw(80) << "*                                  *\n";
    cout << setw(80) << "*                                  *\n";
    cout << setw(80) << "\n";
    cout << "\n\n";
    Sleep(1000);
    system("Pause");

    system("CLS");
    system("Color 0E");
    system("cls");
    int bar1 = 177, bar2 = 219;
    cout << "\n\n\n\t\t\t LOADING...";
    cout << "\n\n\n\t\t\t\t";

    for (int i = 0; i < 40; i++)
        cout << char(bar1);
    cout << "\r";
    cout << "\t\t\t\t";

    for (int i = 0; i < 40; i++)
    {
        cout << char(bar2);
        Sleep(60);
    }
    cout << endl << endl;
    cout << "\t\t\t\t";

    system("Pause");
    system("CLS");
    while (true)
    {

        do {
            flag = false;
            system("Color 0E");
            displayMenu();
            cout << "Enter choice: ";
            cin >> choice;
            system("CLS");

            switch (choice) {
            case '1':
                cout << "Enter username: ";
                cin >> name;
                log_name = name;
                do {
                    cout << "Enter password: ";
                    cin >> password;
                    pass_len = password.length();
                    if (pass_len < 8)
                    {
                        cout << "Password Must be of 8 characters!" << endl;
                    }
                } while (pass_len < 8);

                cout << "Enter city: ";
                cin >> city;
                userGraph.insert(name, password, city);
                break;

            case '2':
                cout << "Enter username: ";
                cin >> name;
                log_name = name;
                cout << "Enter password: ";
                cin >> password;

                flag = userGraph.verify(name, password);
                if (!flag)
                {
                    cout << "Invalid Username or Password!" << endl;
                }
                break;
            case '3':
                cout << "Exiting the Instagram!!!" << endl;
                system("Pause");
                return 0;
            default:
                cout << "Invalid Input!" << endl;
            }
            system("Pause");
            system("CLS");

        } while (!flag);
        system("Color 0E");
        system("cls");
        int bar1 = 177, bar2 = 219;
        cout << "\n\n\n\t\t\t LOADING...";
        cout << "\n\n\n\t\t\t\t";

        for (int i = 0; i < 40; i++)
            cout << char(bar1);
        cout << "\r";
        cout << "\t\t\t\t";

        for (int i = 0; i < 40; i++)
        {
            cout << char(bar2);
            Sleep(60);
        }
        cout << endl << endl;
        cout << "\t\t\t\t";

        system("Pause");
        system("CLS");
        User* foundUser = nullptr;
        do
        {
            system("Color 0E");
            cout << "Instagram ID of: " << log_name << endl;
            display2();
            cout << "Enter Choice: ";
            cin >> choice;
            system("CLS");

            switch (choice)
            {
            case '1':
                userGraph.logout();
                flag = false;
                break;

            case '2':
                cout << "Enter friend's username: ";
                cin >> followName;
                userGraph.sendFollowRequest(followName);
                break;

            case '3':
                cout << "Enter friend's username to accept: ";
                cin >> followName;
                userGraph.acceptFollowRequest(followName);
                break;

            case '4':
                cout << "Enter post content: ";
                cin.ignore();
                getline(cin, content);
                userGraph.getCurrentUser()->posts.push(content);
                cout << "Post added.\n";
                break;

            case '5':
                userGraph.getCurrentUser()->posts.display();
                break;

            case '6': // Send Message
                cout << "Enter recipient's username: ";
                cin >> recipient;
                cout << "Enter message: ";
                cin.ignore();
                getline(cin, message);
                recipientUser = userGraph.searchUser(recipient);
                if (recipientUser) {
                    // Push to the recipient's stack (received message)
                    recipientUser->messages.push(userGraph.getCurrentUser()->name, message);
                    recipientUser->notifications.enqueue(log_name + " sent you a message!");

                    // Push to the sender's stack (sent message)
                    userGraph.getCurrentUser()->messages.push(userGraph.getCurrentUser()->name, message);

                    cout << "Message sent.\n";
                }
                else {
                    cout << "Recipient not found.\n";
                }
                break;

            case '7': {
                cout << "Enter the username of the person whose messages you want to view: ";
                cin >> recipient;
                userGraph.getCurrentUser()->messages.display(userGraph.getCurrentUser()->name, recipient);
                break;
            }

            case '8':
                userGraph.getCurrentUser()->followRequests.display();
                break;

            case '9':
                userGraph.getCurrentUser()->notifications.display();
                break;

            case 'A':
            case 'a':
                userGraph.displayFollowers(userGraph.getCurrentUser()->name);
                break;

            case 'B':
            case 'b':
                userGraph.displayFollowing(userGraph.getCurrentUser()->name);
                break;

            case 'C':
            case 'c':
                cout << "Enter username to search: ";
                cin >> name;
                foundUser = userGraph.searchUser(name);  // Initialize foundUser here
                if (foundUser) {
                    cout << "User found: " << foundUser->name << endl << "City: " << foundUser->city << "\n";
                }
                else {
                    cout << "User not found.\n";
                }
                break;
            case 'D':  // Display search history
                userGraph.displaySearchHistory();
                break;
            case 'E':
                userGraph.displayFollowerPosts();
                break;
            default:
                cout << "Invalid choice. Try again.\n";
            }
            system("Pause");
            system("CLS");
        } while (choice != '1');
    }
    system("Pause");
    return 0;

}
