clone() {

    YELLOW="\e[1;33m"
    CYAN="\e[1;36m"
    BLUE="\e[1;34m"
    GREEN="\e[1;32m"
    RED="\e[1;31m"
    RESET="\e[0m"

    local base="https://github.com"
    local input="$1"

    if [[ -z "$input" ]]; then
        echo -e "${YELLOW}clone usage:${RESET}"
        echo "clone <repo>"
        echo "clone <user>/<repo>"
        return 1
    fi

    local user repo url status target new_name

    if [[ "$input" == */* ]]; then
        user="${input%%/*}"
        repo="${input##*/}"
    else
        user="funbinet"
        repo="$input"
    fi

    url="$base/$user/$repo"
    target="$repo"

    echo -e "${YELLOW}cloning${RESET} ${CYAN}$user${RESET}/${BLUE}$repo${RESET}"

    status=$(curl -s -o /dev/null -w "%{http_code}" "$url")

    if [[ "$status" == "404" ]]; then
        echo -e "${RED}Repo not found${RESET}"
        return 1
    fi

    if [[ "$status" == "403" ]]; then
        echo -e "${RED}Private or denied${RESET}"
        return 1
    fi

    if [[ "$status" != "200" ]]; then
        echo -e "${RED}Error $status${RESET}"
        return 1
    fi

    if [[ -d "$target" ]]; then
         echo -e "${RED}Directory exists:${RESET} ${CYAN}$target${RESET}"

        new_name=$(next_name "$target")
        echo
        echo -e "${BLUE}[Options]${RESET}"
        echo
        echo -e "  ${YELLOW}[1]${RESET} Skip"
        echo -e "  ${YELLOW}[2]${RESET} Rename"
        echo -e "  ${YELLOW}[3]${RESET} Overwrite"
        echo

        read -rp "> " choice

        case "$choice" in
            1)
                echo
                echo -e "${YELLOW}Skipped${RESET}"
                return 0
                ;;

            2)
                mv "$target" "$new_name"
                echo
                echo -e "${CYAN}Renamed to${RESET} ${GREEN}$new_name${RESET}"
                ;;

            3)
                rm -rf "$target"
                echo
                echo -e "${RED}Removed existing folder${RESET}"
                ;;

            *)
                echo
                echo -e "${RED}Invalid option${RESET}"
                return 1
                ;;
        esac
    fi

    git clone "$url.git" "$target" || {
        echo -e "${RED}Clone failed${RESET}"
        return 1
    }

    echo -e "${GREEN}Done${RESET}"
}

next_name() {
    local name="$1"
    local i=1

    if [[ ! -d "$name" ]]; then
        echo "$name"
        return
    fi

    while [[ -d "${name}(${i})" ]]; do
        ((i++))
    done

    echo "${name}(${i})"
}
