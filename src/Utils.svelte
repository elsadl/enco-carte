<script context="module">
  export function capitalizeFirstLetter(string) {
    return string.charAt(0).toUpperCase() + string.slice(1);
  }

  export function capitalize(string) {
    let newString = "";
    let splitString = string.split(" ");
    for (let i = 0; i < splitString.length; i++) {
      newString += splitString[i].charAt(0).toUpperCase();
      newString += splitString[i].slice(1);
      if (i != splitString.length - 1) {
        newString += " ";
      }
    }
    return newString;
  }
</script>