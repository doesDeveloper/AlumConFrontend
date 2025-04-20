<script>
    // Props
    export let username = "";
    export let size = "40px"; // Default size
    export let fontSize = "1rem"; // Default font size
  
    // Function to generate a consistent color for a given username
    function generateColorFromUsername(name) {
      // Simple hash function for the username
      let hash = 0;
      if (name.length === 0) return hash;
      
      for (let i = 0; i < name.length; i++) {
        const char = name.charCodeAt(i);
        hash = ((hash << 5) - hash) + char;
        hash = hash & hash; // Convert to 32bit integer
      }
      
      // Generate hue from hash (0-360)
      const hue = Math.abs(hash % 360);
      
      // Fixed saturation and lightness for readable colors
      return `hsl(${hue}, 70%, 65%)`;
    }
  
    // Get the first letter or default to '?'
    $: firstLetter = username && username.length > 0 ? username.charAt(0).toUpperCase() : '?';
    
    // Generate background color from username
    $: backgroundColor = generateColorFromUsername(username);
  </script>
  
  <div 
    class="user-avatar" 
    style="
      width: {size}; 
      height: {size}; 
      background-color: {backgroundColor};
      font-size: {fontSize};
    "
  >
    {firstLetter}
  </div>
  
  <style>
    .user-avatar {
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 50%;
      color: white;
      font-weight: bold;
      user-select: none;
    }
  </style>