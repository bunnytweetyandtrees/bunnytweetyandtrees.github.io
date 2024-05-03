---
layout: default
---

<script>
  document.addEventListener("DOMContentLoaded", function() {
    var themesRadioButtons = document.querySelectorAll('input[type="radio"][name="themes"]');
    var checkboxesContainer = document.getElementById('checkboxForm'); // Get the checkboxes container
    var wordCellsMap = {}; // Object to store cell indices for each word
    var puzzleSize = 16;
    var themeSelect = document.getElementById('themeSelect');
    var easyRadio = document.getElementById('easyRadio');
    var hardRadio = document.getElementById('hardRadio');

    // Event listener for theme select dropdown
    themeSelect.addEventListener("change", function() {
      var selectedTheme = themeSelect.value;
      var words;
      var puzzles;

      if (easyRadio.checked) {
        words = {{ site.data.puzzle_data.Easy_Placed_Words | jsonify }};
        puzzles = {{ site.data.puzzle_data.Easy_Boards | jsonify }};
      } else if (hardRadio.checked) {
        words = {{ site.data.puzzle_data.Hard_Placed_Words | jsonify }};
        puzzles = {{ site.data.puzzle_data.Hard_Boards | jsonify }};
      }

      var wordList = words[selectedTheme];
      updateCheckboxes(wordList);
      populatePuzzle(puzzles[selectedTheme]);
    });

    // Event listener for difficulty radio buttons
    easyRadio.addEventListener("change", function() {
      themeSelect.dispatchEvent(new Event('change')); // Trigger change event for theme select dropdown
    });

    hardRadio.addEventListener("change", function() {
      themeSelect.dispatchEvent(new Event('change')); // Trigger change event for theme select dropdown
    });

    // Trigger change event for theme select dropdown to load default data
    themeSelect.dispatchEvent(new Event('change'));

    function populatePuzzle(puzzle) {
      var puzzleTable = document.getElementById('puzzleTable');
      puzzleTable.innerHTML = ''; // Clear previous puzzle

      // Iterate through rows and cells to populate puzzle table
      puzzle.forEach(function(row) {
        var tr = document.createElement('tr');
        row.forEach(function(cell) {
          var td = document.createElement('td');
          td.setAttribute('data-letter', cell);
          td.textContent = cell;
          tr.appendChild(td);
        });
        puzzleTable.appendChild(tr);
      });
    }

    function updateCheckboxes(wordList) {
      checkboxesContainer.innerHTML = ''; // Clear previous checkboxes

      // Loop through the wordList and generate checkboxes and labels
      wordList.forEach(function(item, index) {
        var checkbox = document.createElement('input');
        checkbox.type = 'checkbox';
        checkbox.id = 'checkbox' + index;
        checkbox.name = 'checkbox' + index;
        checkbox.setAttribute('data-word', item); // Set data-word attribute
        checkboxesContainer.appendChild(checkbox);

        var label = document.createElement('label');
        label.setAttribute('for', 'checkbox' + index);
        label.textContent = item.toUpperCase();
        checkboxesContainer.appendChild(label);

        var lineBreak = document.createElement('br');
        checkboxesContainer.appendChild(lineBreak);
      });
    }

    // Attach event listeners for checkboxes after they are generated
    checkboxesContainer.addEventListener("change", function(event) {
      if (event.target.type === 'checkbox') {
        var isChecked = event.target.checked;
        var word = event.target.getAttribute("data-word");
        var puzzleTable = document.getElementById('puzzleTable');
        var puzzleCells = puzzleTable.getElementsByTagName("td");

        if (isChecked) {
          markWordInPuzzle(puzzleCells, word);
        } else {
          resetWordInPuzzle(word);
        }
      }
    });

    // Function definitions for marking and resetting words in puzzle...

    // Define your JavaScript function to mark a word in the puzzle
    function markWordInPuzzle(cells, word) {
      console.log("Marking word in puzzle:", word);
      // Convert the word to uppercase and remove non-alphabetic characters
      var cleanWord = word.toUpperCase().replace(/[^A-Z]/g, '');

      var wordFound = false;
      for (var i = 0; i < cells.length; i++) {
        var cell = cells[i];
        if (cell.textContent.toUpperCase() === cleanWord.charAt(0)) {
          var directions = [
            { row: -1, col: 0 }, { row: 1, col: 0 }, // Vertical
            { row: 0, col: -1 }, { row: 0, col: 1 }, // Horizontal
            { row: -1, col: -1 }, { row: -1, col: 1 }, // Diagonal (top left to bottom right)
            { row: 1, col: -1 }, { row: 1, col: 1 } // Diagonal (bottom left to top right)
          ];

          for (var j = 0; j < directions.length; j++) {
            var direction = directions[j];
            var found = checkDirection(cells, cleanWord, cell.parentNode.rowIndex, cell.cellIndex, direction.row, direction.col);
            if (found) {
              wordFound = true;
              break;
            }
          }

          if (wordFound) break;
        }
      }
    }

    function checkDirection(cells, word, row, col, rowDir, colDir) {
      for (var k = 0; k < word.length; k++) {
        var newRow = row + k * rowDir;
        var newCol = col + k * colDir;
        if (newRow < 0 || newRow >= puzzleSize || newCol < 0 || newCol >= puzzleSize) {
          return false;
        }
        var cell = cells[newRow * puzzleSize + newCol];
        if (cell.textContent !== word.charAt(k)) {
          return false;
        }
      }

      var cellIndices = [];
      for (var k = 0; k < word.length; k++) {
        var newRow = row + k * rowDir;
        var newCol = col + k * colDir;
        var cellIndex = newRow * puzzleSize + newCol;
        var cell = cells[cellIndex];
        cell.style.color = "red";
        cell.style.fontWeight = "bold";
        cellIndices.push(cellIndex);
      }
      wordCellsMap[word] = cellIndices;
      console.log(wordCellsMap);
      return true;
    }

    function resetWordInPuzzle(word) {
      // Convert the word to uppercase and remove non-alphabetic characters
      var word = word.toUpperCase().replace(/[^A-Z]/g, '');
      console.log("Resetting word in puzzle:", word);
      var cellIndices = wordCellsMap[word];
      if (!cellIndices) return; // Word not found in map

      for (var i = 0; i < cellIndices.length; i++) {
        var cellIndex = cellIndices[i];
        var cell = document.querySelector('table').getElementsByTagName('td')[cellIndex];
        cell.style.color = ""; // Reset color to default
        cell.style.fontWeight = ""; // Reset font weight to default
      }
      delete wordCellsMap[word]; // Remove word entry from map
      console.log(wordCellsMap);
    }
  });
</script>

<div style="display: flex; justify-content: center; align-items: flex-start;">
  <div style="width: 30%; margin-right: 20px;">
    <h2>Select a Theme</h2>
    <select id="themeSelect">
      {% assign themes = site.data.puzzle_data.Theme %}
      {% for theme in themes %}
        <option value="{{ theme[0] }}">{{ theme[1] }}</option>
      {% endfor %}
    </select>
    <br/>
    <br/>
    <h2>Select the Puzzle</h2>
    <input type="radio" id="easyRadio" name="difficulty" value="Easy" checked>
    <label for="easyRadio">Top (Easy)</label>
    <br/>
    <input type="radio" id="hardRadio" name="difficulty" value="Hard">
    <label for="hardRadio">Bottom (Hard)</label>
    <br/>
    <br/>
    <h2>Words</h2>
    Select a word to see it in the puzzle.
    <form id="checkboxForm" action="">
    </form>
  </div>

  <div style="width: 70%;">
    <h2>Puzzle</h2>
    <table border="1" id="puzzleTable">
      <!-- Puzzle will be dynamically populated here -->
    </table>
  </div>

</div>
