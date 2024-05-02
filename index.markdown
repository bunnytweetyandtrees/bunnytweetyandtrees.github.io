---
layout: default
---

<script>
  document.addEventListener("DOMContentLoaded", function() {
    var checkboxes = document.querySelectorAll('input[type="checkbox"]');

    checkboxes.forEach(function(checkbox) {
      checkbox.addEventListener("change", function() {
        console.log("Checkbox changed:", this.checked);
        var word = this.getAttribute("data-word");
        console.log("Word:", word);
        var puzzleTable = document.querySelector("table");
        var puzzleCells = puzzleTable.getElementsByTagName("td");

        if (this.checked) {
          markWordInPuzzle(puzzleCells, word);
        } else {
          resetWordInPuzzle(puzzleCells, word);
        }
      });
    });

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
        if (newRow < 0 || newRow >= 16 || newCol < 0 || newCol >= 16) {
          return false;
        }
        var cell = cells[newRow * 16 + newCol];
        if (cell.textContent !== word.charAt(k)) {
          return false;
        }
      }

      for (var k = 0; k < word.length; k++) {
        var newRow = row + k * rowDir;
        var newCol = col + k * colDir;
        var cell = cells[newRow * 16 + newCol];
        cell.style.color = "red";
        cell.style.fontWeight = "bold";
      }
      return true;
    }


    function resetWordInPuzzle(cells, word) {
      console.log("Resetting word in puzzle:", word);
      for (var i = 0; i < cells.length; i++) {
        var cell = cells[i];
        if (cell.style.color === "red" && cell.style.fontWeight === "bold") {
          cell.style.color = ""; // Reset color to default
          cell.style.fontWeight = ""; // Reset font weight to default
        }
      }
    }

  });
</script>

<div style="display: flex; justify-content: space-between;">
  <div style="width: 30%; margin-right: 20px;"> <!-- Added margin-right for the gap -->
    <h2>Checkboxes</h2>

    <form action="">
      {% assign total_words = site.data.words | size %}
      {% for item in site.data.words limit: total_words %}
        <input type="checkbox" id="checkbox{{ item.id }}" name="checkbox{{ item.id }}" data-word="{{ item.label }}">
        <label for="checkbox{{ item.id }}">{{ item.label }}</label><br>
      {% endfor %}
    </form>

    Select a word to see it in the puzzle.

  </div>

  <div style="width: 70%;">
    <h2>Word Search Puzzle</h2>
    <table border="1">
      {% assign puzzle = site.data.puzzle.puzzle %}
      {% for row in puzzle %}
        <tr>
          {% for cell in row %}
            <td data-letter="{{ cell }}">{{ cell }}</td>
          {% endfor %}
        </tr>
      {% endfor %}
    </table>
  </div>
</div>
