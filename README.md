
```com.Termux```

```termux-setup-storage```

```
find /sdcard/ -name "rish_shizuku.dex" -exec cp {} ~ \; 2>/dev/null
find /sdcard/ -name "rish" -type f -exec cp {} ~ \; 2>/dev/null
cd ~
```
```chmod +x rish```
```./rish```

```nano optimize.sh```

```
#!/system/bin/sh

# Define Color Codes
G='\033[1;32m'
R='\033[1;31m'
C='\033[1;36m'
Y='\033[1;33m'
W='\033[1;37m'
N='\033[0m'

# Handle Ctrl+C gracefully
trap 'printf "\n${R}[!] Script interrupted by user! Exiting safely...${N}\n"; rm -f /data/local/tmp/.opt_status /data/local/tmp/.opt_detail 2>/dev/null; exit 1' INT TERM

# Function to check device temperature for safety
check_thermal() {
    local temp=$(dumpsys battery | grep "temperature" | awk '{print $2}')
    if [ ! -z "$temp" ]; then
        # Android reports temp in tenths of a degree Celsius (e.g., 400 = 40°C)
        if [ "$temp" -gt 430 ]; then
            local current_c=$((temp / 10))
            printf "${R}[CRITICAL WARNING] Device is too hot (${current_c}°C)!${N}\n"
            printf "${Y}Aborting heavy compilation to prevent hardware damage.${N}\n"
            exit 1
        fi
    fi
}

# Ultra-Accurate Spin function (Enhanced with Deep Verification)
spin() {
    local pid=$1
    local msg=$2
    local success_msg=${3:-"Done!"}
    local delay=0.1
    local spinstr='|/-\ '
    local temp
    local status
    
    # Loop while background process is alive
    while kill -0 "$pid" 2>/dev/null; do
        temp=${spinstr#?}
        printf "\r${Y}[%c]${N} %s" "$spinstr" "$msg"
        spinstr=$temp${spinstr%"$temp"}
        sleep $delay
    done
    
    # Read the real execution status from local tmp
    if [ -f /data/local/tmp/.opt_status ]; then
        status=$(cat /data/local/tmp/.opt_status)
        rm -f /data/local/tmp/.opt_status
    else
        status=1
    fi
    
    # Detailed verification message based on status code
    if [ "$status" -eq 0 ]; then
        if [ -f /data/local/tmp/.opt_detail ]; then
            local detail=$(cat /data/local/tmp/.opt_detail)
            rm -f /data/local/tmp/.opt_detail
            printf "\r${G}[✔]${N} %s - ${G}%s${N} (%s)\n" "$msg" "$success_msg" "$detail"
        else
            printf "\r${G}[✔]${N} %s - ${G}%s${N}\n" "$msg" "$success_msg"
        fi
    elif [ "$status" -eq 143 ]; then
        printf "\r${R}[✘]${N} %s - ${R}Skipped/Unsupported by OS Profile!${N}\n" "$msg"
    else
        printf "\r${R}[✘]${N} %s - ${R}Failed! (OS Error Status: $status)${N}\n" "$msg"
    fi
}

clear
printf "${C}==========================================${N}\n"
printf "${G}      Android Core Optimizer (Pro)        ${N}\n"
printf "${C}==========================================${N}\n\n"

# Permission Check
USER_ID=$(id -u)
if [ "$USER_ID" != "0" ] && [ "$USER_ID" != "2000" ]; then
    printf "${R}[ERROR] Access Denied!${N}\n"
    printf "${Y}This script requires ADB, Shizuku, or Root privileges.${N}\n"
    exit 1
fi

# Display Menu (Completely Separated Tasks)
printf "${W}Select an optimization method:${N}\n"
printf "${C}[1]${N} Target Specific App (Speed-Profile)\n"
printf "${C}[2]${N} Target All Apps (Background Dexopt)\n"
printf "${C}[3]${N} Force Compile All Apps (Speed-Profile - Requires Root/Supported ADB)\n"
printf "${C}[4]${N} Clear System Caches Only\n\n"

printf "Enter choice [1-4]: "
read -r choice
printf "\n"

case "$choice" in
    "1")
        printf "${Y}Enter target package name: ${N}"
        read -r pkg
        
        if [ -z "$pkg" ]; then
            printf "\n${R}[ERROR] Package name cannot be empty!${N}\n"
            exit 1
        fi

        case "$pkg" in
            *[!a-zA-Z0-9_.]*)
                printf "\n${R}[ERROR] Invalid format!${N}\n"
                exit 1
                ;;
        esac

        if ! pm list packages 2>/dev/null | grep -qFx "package:$pkg"; then
            printf "\n${R}[ERROR] Package '$pkg' not found!${N}\n"
            exit 1
        fi

        printf "\n${C}[1/1] Compiling package: ${pkg}...${N}\n"
        sh -c '
            (pm compile -m speed-profile -f "'"$pkg"'" || cmd package compile -m speed-profile -f "'"$pkg"'") >/dev/null 2>&1
            RET=$?
            if [ $RET -eq 0 ]; then
                if dumpsys package "'"$pkg"'" | grep -q "speed-profile"; then
                    echo "Verified: speed-profile" > /data/local/tmp/.opt_detail
                    echo 0 > /data/local/tmp/.opt_status
                else
                    echo "Skipped: Missing Profile Data" > /data/local/tmp/.opt_detail
                    echo 143 > /data/local/tmp/.opt_status
                fi
            else
                echo $RET > /data/local/tmp/.opt_status
            fi
        ' &
        spin $! "Optimizing $pkg" "Verified!"
        ;;
        
    "2")
        printf "${W}Safe Mode selected.${N}\n"
        printf "${Y}Press [ENTER] to confirm...${N}"
        read -r dummy
        
        printf "\n${C}[1/1] Triggering Background Dexopt...${N}\n"
        sh -c '
            (pm bg-dexopt-job || cmd package bg-dexopt-job || cmd jobscheduler run -f android 2618) >/dev/null 2>&1
            echo $? > /data/local/tmp/.opt_status
        ' &
        spin $! "Triggering Background Task" "Dispatched to OS Queue!"
        ;;

    "3")
        printf "${R}[WARNING] This will force compile ALL apps immediately!${N}\n"
        check_thermal
        printf "${Y}Your device might get warm. Press [ENTER] to confirm...${N}"
        read -r dummy
        
        printf "\n${C}[1/1] Compiling ALL packages (Speed-Profile)...${N}\n"
        printf "${Y}(This process may take 5-15 minutes, please wait...)${N}\n"
        
        sh -c '
            (pm compile -a -m speed-profile || cmd package compile -a -m speed-profile) >/dev/null 2>&1
            RET=$?
            echo $RET > /data/local/tmp/.opt_status
        ' &
        spin $! "Optimizing All Apps" "Completely Dispatched!"
        ;;
        
    "4")
        printf "\n${C}[1/1] Clearing all app caches...${N}\n"
        sh -c '
            pm trim-caches 999G >/dev/null 2>&1
            echo $? > /data/local/tmp/.opt_status
        ' &
        spin $! "Clearing System Caches" "Done!"
        ;;
        
    *)
        printf "${R}[ERROR] Invalid option!${N}\n"
        exit 1
        ;;
esac

printf "\n${G}==========================================${N}\n"
printf "${G}   System Optimization Completed!         ${N}\n"
printf "${G}==========================================${N}\n"
```

```chmod +x optimize.sh```
```optimize.sh /sdcard/Download/```

```chmod +x ~/rish && ./rish```

```sh /sdcard/Download/optimize.sh```

```./rish```

```exit```
