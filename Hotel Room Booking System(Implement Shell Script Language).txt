NUMBER_OF_HOTEL_ROOMS=100

declare -a rooms
declare -A bookings

initialize_rooms() {
  for ((i=0; i<NUMBER_OF_HOTEL_ROOMS; i++)); do
    rooms[$i]=0
  done
}

display_room_status() {
  echo "Hotel Room Status:"
  for ((i=0; i<NUMBER_OF_HOTEL_ROOMS; i++)); do
    echo "Room $((i+1)): ${rooms[$i]} "
  done
  echo
}

is_room_available() {
  local avail_room=$1
  if [[ ${rooms[$avail_room]} -eq 0 ]]; then
    return 0
  else
    return 1
  fi
}

book_room() {
  local avail_room=$1
  local name=$2
  if [[ ${rooms[$avail_room]} -eq 0 ]]; then
    rooms[$avail_room]=1
    bookings[$avail_room]=$name
    echo "This Room $((avail_room+1)) has been booked by $name."
  else
    echo "This Room $((avail_room+1)) is already occupied."
  fi
}

release_room() {
  local avail_room=$1
  if [[ ${rooms[$avail_room]} -eq 1 ]]; then
    rooms[$avail_room]=0
    local name=${bookings[$avail_room]}
    unset bookings[$avail_room]
    echo "This Room $((avail_room+1)) is booked by $name. Now It has been released."
  else
    echo "This Room $((avail_room+1)) is already unoccupied."
  fi
}

view_bookings() {
  if [[ ${#bookings[@]} -eq 0 ]]; then
    echo "No bookings found."
  else
    echo "Room Bookings:"
    for avail_room in "${!bookings[@]}"; do
      local name=${bookings[$avail_room]}
      echo "Room $((avail_room+1)): $name"
    done
  fi
}

search_by_customer_name() {
  local name=$1
  local found=false
  echo "Search Results for Guest Name '$name':"
  for avail_room in "${!bookings[@]}"; do
    local booked_name=${bookings[$avail_room]}
    if [[ $booked_name == $name ]]; then
      echo "Room $((avail_room+1))"
      found=true
    fi
  done
  if [[ $found == false ]]; then
    echo "No bookings found for Guest name '$name'."
  fi
}

search_by_room_number() {
  local avail_room=$1
  if validate_room_number "$avail_room"; then
    local name=${bookings[$((avail_room-1))]}
    if [[ -n $name ]]; then
      echo "Booking Details for Room Number $avail_room:"
      echo "Guest Name: $name"
    else
      echo "Room $avail_room is not booked."
    fi
  else
    echo "Invalid room number."
  fi
}

calculate_revenue() {
  local revenue=0
  for avail_room in "${!bookings[@]}"; do
    ((revenue++))
  done
  echo "Total Room Revenue: $revenue"
}

check_room_availability() {
  local avail_room=$1
  if validate_room_number "$avail_room"; then
    if is_room_available $((avail_room-1)); then
      echo "Room $avail_room is available."
    else
      echo "Room $avail_room is occupied."
    fi
  else
    echo "Invalid room number."
  fi
}

view_unbooked_rooms() {
  echo "Unbooked Rooms:"
  local unbooked_rooms=()
  for avail_room in "${!rooms[@]}"; do
    if is_room_available $avail_room; then
      unbooked_rooms+=("$((avail_room+1))")
    fi
  done
  if [[ ${#unbooked_rooms[@]} -eq 0 ]]; then
    echo "All rooms are booked."
  else
    for avail_room in "${unbooked_rooms[@]}"; do
      echo "Room $avail_room"
    done
  fi
}

menu() {
  echo -e "\n....................Hotel Room Management System....................\n\n"
  echo -e "!!!...Hotel Relax...!!!"
  echo -e "\n1. Display All Room Status"
  echo -e "\n2. Book a Room"
  echo -e "\n3. Release a Room"
  echo -e "\n4. View All Booking Rooms"
  echo -e "\n5. Search by Guest Name"
  echo -e "\n6. Search by Room Number"
  echo -e "\n7. Calculate Total Revenue"
  echo -e "\n8. Check Room Availability"
  echo -e "\n9. View All Unbooked Rooms"
  echo -e "\n10. Exit\n"
}

validate_room_number() {
  local room_num=$1
  if (( room_num >= 1 && room_num <= NUMBER_OF_HOTEL_ROOMS )); then
    return 0
  else
    return 1
  fi
}

initialize_rooms

while true; do
  menu
  read -p "Enter your choice: " choice

  case $choice in
    1)
      display_room_status
      ;;
    2)
      read -p "Enter the room number to book: " room_num
      if validate_room_number "$room_num"; then
        is_room_available $((room_num-1))
        if [[ $? -eq 0 ]]; then
          read -p "Enter the Guest name: " customer_name
          book_room $((room_num-1)) "$customer_name"
        else
          echo "This Room $room_num is not available."
        fi
      else
        echo "Invalid room number."
      fi
      ;;
    3)
      read -p "Enter the room number to release: " room_num
      if validate_room_number "$room_num"; then
        release_room $((room_num-1))
      else
        echo "Invalid room number."
      fi
      ;;
    4)
      view_bookings
      ;;
    5)
      read -p "Enter the Guest name to search: " search_name
      search_by_customer_name "$search_name"
      ;;
    6)
      read -p "Enter the room number to search: " search_room_num
      search_by_room_number "$search_room_num"
      ;;
    7)
      calculate_revenue
      ;;
    8)
      read -p "Enter the room number to check availability: " room_num
      check_room_availability "$room_num"
      ;;
    9)
      view_unbooked_rooms
      ;;
    10)
      echo "Exiting the Management..."
      exit 0
      ;;
    *)
      echo "Invalid choice. Please try again."
      ;;
  esac
done
