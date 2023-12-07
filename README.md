# fileHandling
File handling (read and write) in c++

#include <iostream>
#include <string>
#include <fstream>
#include <iomanip>
#include <limits>
#include <algorithm>
using namespace std;

const int MAX_TOPPINGS = 2;
const int MAX_DONUTS = 50;

struct donutType {
	string name;
	bool type;
	double price;
	string filling;
	string toppings[MAX_TOPPINGS];
};

ifstream getFileStream(string);
int getDonuts(ifstream&, donutType[]);
bool continueMenu(string);
void sortByPrice(donutType[], int);
int searchByName(const donutType[], int, string);
void removeDonutFromList(donutType[], int&, int);
int getCheapestDonut(const donutType[], int);
void soldDonut(donutType, donutType[], int&);
void outputSoldDonuts(ofstream&, const donutType[], int);
void displayAvailableDonuts(const donutType[], int);
string allCaps(string);

/**
 * The main function to manage the donut ordering program.
 *
 * @return The exit status of the program.
 */
int main() {
	ifstream infile;
	ofstream outfile("sold.csv");
	string filename;
	string request;
	donutType donuts[MAX_DONUTS];
	donutType soldDonuts[MAX_DONUTS];
	int amtDonuts = 0;
	int amtSold = 0;
	double total = 0;

	cout << fixed << setprecision(2);

	// gets step 1 the input stream
	infile = getFileStream("Enter a donut file: ");

	// step 2 get total donuts
	amtDonuts = getDonuts(infile, donuts);

	// step 3 print to console
	cout << "Welcome to Hank's Donut World!\n\n";
	while (true) {

		// step 4 print the available donuts to console
		displayAvailableDonuts(donuts, amtDonuts);

		//step 5 ask for user input
		string nameEnteredByUser;
		cout << "Enter donut name or cheapest: ";
		getline(cin, nameEnteredByUser);

		// step 6 check the user input choice and call function accordingly
		int donutIndex;
		transform(nameEnteredByUser.begin(), nameEnteredByUser.end(),
				nameEnteredByUser.begin(), ::tolower);

		if (nameEnteredByUser.compare("cheapest") == 0)
			donutIndex = getCheapestDonut(donuts, amtDonuts);
		else
			donutIndex = searchByName(donuts, amtDonuts, nameEnteredByUser);

		// step 7 check if the search results was empty
		if (donutIndex == -1) {
			cout << "Donut not found!\n";
			continue;
		}

		// step 8 print the selection to console
		cout << "You selected " << donuts[donutIndex].name
				<< ".\nExcellent choice!\n";

		// Step 9 inserting the sold donut into solddonuts array
		total += donuts[donutIndex].price;
		soldDonuts[amtSold] = donuts[donutIndex];
		amtSold++;

		// Step 10 removing the donut from donuts array
		removeDonutFromList(donuts, amtDonuts, donutIndex);

		// Step 11 choice to continue with more purchase
		bool complete = continueMenu("Will this complete your order? ");
		if (!complete) {
			if (amtDonuts > 0)
				continue; // go to Step 4
			else
				break; // End the program if there are no available donuts left

		}

		// Step 13 sort the donuts sold by price using bubble sort
		sortByPrice(soldDonuts, amtSold);

		// Step 14
		if (outfile.is_open()) {
			outfile << "Sold," << fixed << setprecision(2) << total << "\n";
			outputSoldDonuts(outfile, soldDonuts, amtSold);
		} else {
			cerr << "Error opening output file.\n";
		}
		break; // End the program
	}

	return 0;
}

/**
 * Retrieves an input file stream for a given filename after prompting the user.
 *
 * @param msg The message to prompt the user for the filename.
 * @return An ifstream object for the specified filename.
 */
ifstream getFileStream(string msg) {
	ifstream fileStream;
	string fileName;

	while (true) {
		// Prompt user for a filename
		cout << msg;
		getline(cin, fileName);

		// Open the file stream
		fileStream.open(fileName);

		// Check if the file stream is open
		if (fileStream.is_open()) {
			break;
		} else {
			fileStream.clear(); // Clear any error flags
			cin.ignore(numeric_limits<streamsize>::max(), '\n'); // Clear input buffer
		}
	}

	return fileStream;
}

/**
 * Reads donut data from an input file stream into a struct array of donutType.
 *
 * @param infile An input file stream containing donut data.
 * @param donuts An array of donutType to store the read data.
 * @return The number of donuts read from the file.
 */
int getDonuts(ifstream &infile, donutType donuts[]) {
	const int MAX_RECORDS = 50;
	size_t MAX_CHARS = 100;
	char line[MAX_CHARS];
	int count = 0;

	// Read and ignore the header line
	infile.getline(line, MAX_CHARS);

	// Read the CSV file line by line
	while (!infile.fail() && count < MAX_RECORDS) {
		// Read the entire line into 'line'
		infile.getline(line, MAX_CHARS);

		// Create a strings to parse the line
		string lineString(line);
		if(lineString.size() < 2)
			break;
		// Tokenize the line based on commas
		size_t start = 0;
		size_t end = lineString.find(',');

		// Read name
		donuts[count].name = lineString.substr(0, end);
		//add plus one to skip the comma
		lineString = lineString.substr(end+1);

		// Read type
		end = lineString.find(',');
		donuts[count].type = (lineString.substr(0, end) == "Cake");
		lineString = lineString.substr(end+1);

		// Read filling
		end = lineString.find(',');
		donuts[count].filling = lineString.substr(0, end);
		lineString = lineString.substr(end+1);

		// Read toppings
		end = lineString.find(',');
		donuts[count].toppings[0] = lineString.substr(0, end);
		lineString = lineString.substr(end+1);

		end = lineString.find(',') ;
		donuts[count].toppings[1] = lineString.substr(0, end);
		lineString = lineString.substr(end+1);

		// Read price
		donuts[count].price = stod(lineString);

		count++;
	}
	return count;
}

/**
 * Displays a prompt and waits for user input to continue or exit.
 *
 * @param prompt The message to display as a prompt.
 * @return True if the user chooses to continue, false if the user chooses to exit.
 */
bool continueMenu(string prompt) {
	string input;

	while (true) {
		cout << prompt;
		getline(cin, input);

		// Convert the input to lowercase for case-insensitive comparison
		transform(input.begin(), input.end(), input.begin(), ::tolower);

		if (input == "no")
			return false;
		else if (input == "yes")
			return true;
		else
			cerr << "Invalid input. Please enter 'Yes' or 'No' (case insensitive).\n";

	}
}

/**
 * Sorts an array of donuts based on their prices in ascending order using bubble sort.
 *
 * @param donuts An array of donuts to be sorted.
 * @param amtDonuts The number of donuts in the array.
 */
void sortByPrice(donutType donuts[], int amtDonuts) {
	for (int i = 0; i < amtDonuts - 1; ++i) {
		for (int j = 0; j < amtDonuts - i - 1; ++j) {
			// Compare prices and swap if needed
			if (donuts[j].price > donuts[j + 1].price) {
				// Swap
				donutType temp = donuts[j];
				donuts[j] = donuts[j + 1];
				donuts[j + 1] = temp;
			}
		}
	}
}

/**
 * Searches for a donut by name in an array of donuts.
 *
 * @param donuts An array of donuts to be searched.
 * @param amtDonuts The number of donuts in the array.
 * @param name The name of the donut to be searched.
 * @return The index of the found donut if present, otherwise -1.
 */
int searchByName(const donutType donuts[], int amtDonuts, string name) {

	transform(name.begin(), name.end(), name.begin(), ::tolower);

	for (int i = 0; i < amtDonuts; ++i) {
		string donutName = donuts[i].name;
		transform(donutName.begin(), donutName.end(), donutName.begin(),
				::tolower);
		if (donutName.compare(name) == 0)
			return i; // Return the index if the name is found
	}
	return -1; // Return -1 if the name is not found
}

/**
 * Finds the index of the cheapest donut in an array of donuts.
 *
 * @param donuts An array of donuts to be searched.
 * @param amtDonuts The number of donuts in the array.
 * @return The index of the cheapest donut if the array is not empty, otherwise -1.
 */
int getCheapestDonut(const donutType donuts[], int amtDonuts) {
	if (amtDonuts <= 0)
		return -1; // Return -1 if the array is empty

	// Initialize to maximum possible value
	double minPrice = donuts[0].price;
	// Index of the cheapest donut
	int minIndex = 0;

	for (int i = 1; i < amtDonuts; ++i) {
		if (donuts[i].price < minPrice) {
			minPrice = donuts[i].price;
			minIndex = i;
		}
	}

	return minIndex;
}

/**
 * Converts a given string to uppercase.
 *
 * @param s The input string to be converted to uppercase.
 * @return The uppercase version of the input string.
 */
string allCaps(string s) {
	string upper = s;

	for (char &c : upper)
		c = toupper(static_cast<unsigned char>(c));

	return upper;
}

/**
 * Removes a donut from the list based on the provided index.
 *
 * @param donuts The array of donuts.
 * @param amtDonuts The current number of donuts in the array.
 * @param removeIndex The index of the donut to be removed.
 */
void removeDonutFromList(donutType donuts[], int &amtDonuts, int removeIndex) {
	if (removeIndex < 0 || removeIndex >= amtDonuts) {
		cerr << "Invalid index to remove. Index out of range." << endl;
		return;
	}

	// Shift elements to fill the gap
	for (int i = removeIndex; i < amtDonuts - 1; ++i) {
		donuts[i] = donuts[i + 1];
	}

	// Decrement the amount of donuts
	amtDonuts--;
}

/**
 * Outputs information about sold donuts to an output file.
 *
 * @param outfile The output file stream.
 * @param soldDonuts The array of sold donuts.
 * @param amtSold The current number of sold donuts in the array.
 */
void outputSoldDonuts(ofstream &outfile, const donutType soldDonuts[],
		int amtSold) {

	if (!outfile.is_open()) {
		cerr << "Error: Output file is not open." << endl;
		return;
	}

	outfile << fixed << setprecision(2);
	outfile << "Name,Type,Filling,Topping1,Topping2,Price" << endl;

	for (int i = 0; i < amtSold; ++i) {
		outfile << soldDonuts[i].name << ",";
		outfile << (soldDonuts[i].type ? "Cake" : "Dough") << ",";
		outfile << soldDonuts[i].filling << ",";
		outfile << soldDonuts[i].toppings[0] << ",";
		outfile << soldDonuts[i].toppings[1] << ",";
		outfile << soldDonuts[i].price << "\n";
	}
}

/**
 * Displays the list of available donuts with their names and prices.
 *
 * @param donuts The array of available donuts.
 * @param amtDonuts The current number of available donuts in the array.
 */
void displayAvailableDonuts(const donutType donuts[], int amtDonuts) {
	cout << "List of donuts" << endl;
	cout << "---------------------------" << endl;
	int count = 0;
	for (int i = 0; i < amtDonuts; ++i)
		cout << donuts[i].name << " " << donuts[i].price << endl;

	cout << endl;
}
