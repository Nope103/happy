-- ========== BETANOT fx-880 PRO - GIẢ LẬP MÁY TÍNH CASIO ==========
local history = {}
local Ans = 0
local angleMode = "DEG"

-- Giao diện đầu trang kiểu Casio
function casioHeader(mode)
    print("┌───────────────────────────────┐")
    print("│  BETANOT VICTORIA 6.0 PRO     │")
    print("│  [MODE: " .. mode .. "]   " .. angleMode .. "         │")
    print("└───────────────────────────────┘")
end

-- Chuyển x^y thành math.pow(x,y)
function fixExponent(expr)
    return expr:gsub("(%d+%.?%d*)%s*%^%s*([%-%d%.]+)", "math.pow(%1,%2)")
end

-- Chuyển độ sang rad nếu cần
function degToRad(x)
    return angleMode == "DEG" and x * math.pi / 180 or x
end

-- Tiền xử lý biểu thức
function preprocess(expr)
    expr = expr
        :gsub("Ans", tostring(Ans))
        :gsub("pi", "math.pi")
        :gsub("sqrt", "math.sqrt")
        :gsub("log", "math.log")
        :gsub("exp", "math.exp")
        :gsub("(%d+%.?%d*)%%", "(%1/100)")

    expr = expr:gsub("sin%((.-)%)", function(x) return "math.sin(degToRad("..x.."))" end)
    expr = expr:gsub("cos%((.-)%)", function(x) return "math.cos(degToRad("..x.."))" end)
    expr = expr:gsub("tan%((.-)%)", function(x) return "math.tan(degToRad("..x.."))" end)

    return fixExponent(expr)
end

-- Tính toán biểu thức
function calculate(expr)
    local processed = preprocess(expr)
    local f, err = load("return " .. processed)
    if not f then return nil, err end
    local ok, result = pcall(f)
    if ok then
        Ans = result
        return result
    else
        return nil, result
    end
end
-- Giải phương trình bằng chia đôi
function solveForX(equation)
    local left, right = equation:match("^(.-)%s*=%s*(.-)$")
    if not left or not right then return nil, "Phương trình không hợp lệ" end

    local function f(x)
        local le = fixExponent(preprocess(left:gsub("x", "("..x..")")))
        local ri = fixExponent(preprocess(right:gsub("x", "("..x..")")))
        local l = calculate(le)
        local r = calculate(ri)
        if not l or not r then return nil end
        return l - r
    end

    local a, b = -1000, 1000
    local eps = 1e-6
    local steps = 2000
    local last_x, last_y = nil, nil

    for i = 0, steps do
        local x = a + (b - a) * i / steps
        local y = f(x)
        if y and last_y and y * last_y < 0 then
            local low, high = last_x, x
            for _ = 1, 100 do
                local mid = (low + high) / 2
                local fmid = f(mid)
                if not fmid then break end
                if math.abs(fmid) < eps then return mid end
                if fmid * f(low) < 0 then
                    high = mid
                else
                    low = mid
                end
            end
            return (low + high) / 2
        end
        last_x = x
        last_y = y
    end

    return nil, "Không tìm thấy nghiệm thực trong khoảng [-1000, 1000]"
end

-- Giai thừa
function factorial(n)
    if n < 0 then return nil end
    local result = 1
    for i = 2, n do
        result = result * i
    end
    return result
end

-- Tính xác suất lớp 8
function probabilityMenu()
    print("1. Hoán vị P(n, k)")
    print("2. Chỉnh hợp A(n, k)")
    print("3. Tổ hợp C(n, k)")
    io.write("Chọn loại phép toán: ")
    local opt = io.read()
    io.write("Nhập n: ")
    local n = tonumber(io.read())
    io.write("Nhập k: ")
    local k = tonumber(io.read())

    if not n or not k or n < 0 or k < 0 or k > n then
        print("⚠️ Dữ liệu không hợp lệ.")
        return
    end

    if opt == "1" or opt == "2" then
        local result = factorial(n) / factorial(n - k)
        print("Kết quả: P(" .. n .. ", " .. k .. ") = " .. result)
    elseif opt == "3" then
        local result = factorial(n) / (factorial(k) * factorial(n - k))
        print("Kết quả: C(" .. n .. ", " .. k .. ") = " .. result)
    else
        print("⚠️ Lựa chọn không hợp lệ.")
    end
end

-- Ghi lịch sử vào file
function saveToFile(entry)
    local f = io.open("history.txt", "a")
    f:write(entry .. "\n")
    f:close()
end

-- Xem lịch sử
function viewHistory()
    print("🧾 LỊCH SỬ TÍNH TOÁN:")
    if #history == 0 then
        print("Chưa có phép tính nào.")
    else
        for i, line in ipairs(history) do
            print(i .. ". " .. line)
        end
    end
end

-- Đổi chế độ góc
function switchAngleMode()
    angleMode = (angleMode == "DEG") and "RAD" or "DEG"
    print("📐 Chuyển sang chế độ góc: " .. angleMode)
end

-- Vẽ đồ thị (ghi dữ liệu vào file)
function exportGraphData()
    io.write("Nhập biểu thức theo x (VD: x^2 + 2*x + 1): ")
    local expr = io.read()
    io.write("Nhập x_min: ")
    local xmin = tonumber(io.read())
    io.write("Nhập x_max: ")
    local xmax = tonumber(io.read())
    io.write("Nhập bước (step): ")
    local step = tonumber(io.read())

    local file = io.open("graph_data.txt", "w")
    for x = xmin, xmax, step do
        local fx_expr = preprocess(expr:gsub("x", "("..x..")"))
        local f = load("return " .. fx_expr)
        local ok, y = pcall(f)
        if ok and y then
            file:write(x .. "," .. y .. "\n")
        end
    end
    file:close()
    print("✅ Dữ liệu đồ thị đã lưu vào 'graph_data.txt'")
end

-- Đổi đơn vị
function unitConverter()
    print("1. Mét → cm, km, inch")
    print("2. Kg → g, lb")
    print("3. Độ ↔ radian")
    io.write("Chọn loại chuyển đổi: ")
    local opt = io.read()
    io.write("Nhập giá trị: ")
    local val = tonumber(io.read())

    if opt == "1" then
        print(val .. " m = " .. (val * 100) .. " cm")
        print(val .. " m = " .. (val / 1000) .. " km")
        print(val .. " m = " .. (val * 39.3701) .. " inch")
    elseif opt == "2" then
        print(val .. " kg = " .. (val * 1000) .. " g")
        print(val .. " kg = " .. (val * 2.20462) .. " lb")
    elseif opt == "3" then
        print(val .. " độ = " .. (val * math.pi / 180) .. " rad")
        print(val .. " rad = " .. (val * 180 / math.pi) .. " độ")
    else
        print("⚠️ Không hợp lệ.")
    end
end

-- Công thức hình học
function geometryFormulas()
    print("\n📐 CÔNG THỨC HÌNH HỌC CƠ BẢN:")
    print("- Chu vi hình tròn: C = 2 * pi * r")
    print("- Diện tích hình tròn: S = pi * r^2")
    print("- Chu vi hình chữ nhật: P = 2 * (a + b)")
    print("- Diện tích hình chữ nhật: S = a * b")
    print("- Chu vi hình tam giác: P = a + b + c")
    print("- Diện tích tam giác: S = (a * h) / 2")
    print("- Diện tích tam giác (Heron): S = sqrt(p*(p-a)*(p-b)*(p-c)), p = (a+b+c)/2")
    print("- Diện tích hình thang: S = ((a + b) * h) / 2")
    print("- Thể tích hình lập phương: V = a^3")
    print("- Thể tích hình hộp chữ nhật: V = a * b * c")
    print("- Thể tích hình trụ: V = pi * r^2 * h\n")
end

-- Hằng đẳng thức đáng nhớ
function algebraIdentities()
    print("\n🧠 HẰNG ĐẲNG THỨC ĐÁNG NHỚ:")
    print("- (a + b)^2 = a^2 + 2ab + b^2")
    print("- (a - b)^2 = a^2 - 2ab + b^2")
    print("- (a + b)(a - b) = a^2 - b^2")
    print("- a^3 + b^3 = (a + b)(a^2 - ab + b^2)")
    print("- a^3 - b^3 = (a - b)(a^2 + ab + b^2)")
    print("- (x + y + z)^2 = x^2 + y^2 + z^2 + 2xy + 2yz + 2zx\n")
end

-- Giao diện menu chính
function menu()
    while true do
        casioHeader("CALC / EQN")
        print("1. Tính toán biểu thức (CALC)")
        print("2. Giải phương trình (EQN)")
        print("3. Xem lịch sử")
        print("4. Đổi chế độ góc (DEG/RAD)")
        print("5. Vẽ đồ thị (xuất file)")
        print("6. Chuyển đổi đơn vị")
        print("7. Tính xác suất (lớp 8)")
        print("8. Thoát")
        print("9. Công thức hình học")
        print("10. Hằng đẳng thức đáng nhớ")
        io.write("➤ Chọn chức năng (1-10): ")
        local choice = io.read()

        if choice == "1" then
            io.write("Nhập biểu thức (VD: sqrt(16) + sin(30)): ")
            local expr = io.read()
            local result, err = calculate(expr)
            local time = os.date("[%Y-%m-%d %H:%M:%S]")
            if result then
                local entry = time .. " " .. expr .. " = " .. result
                print("✅ " .. entry)
                table.insert(history, entry)
                saveToFile(entry)
            else
                print("❌ Lỗi: " .. err)
            end
        elseif choice == "2" then
            io.write("Nhập phương trình (VD: x^3 - 6*x^2 + 11*x - 6 = 0): ")
            local eqn = io.read()
            local result, err = solveForX(eqn)
            local time = os.date("[%Y-%m-%d %H:%M:%S]")
            if result then
                local entry = time .. " " .. eqn .. " ⇒ x = " .. string.format("%.6f", result)
                print("✅ " .. entry)
                table.insert(history, entry)
                saveToFile(entry)
            else
                print("❌ Lỗi: " .. err)
            end
        elseif choice == "3" then
            viewHistory()
        elseif choice == "4" then
            switchAngleMode()
        elseif choice == "5" then
            exportGraphData()
        elseif choice == "6" then
            unitConverter()
        elseif choice == "7" then
            probabilityMenu()
        elseif choice == "8" then
            print("👋 Tạm biệt! Cảm ơn bạn đã sử dụng BETANOT VICTORIA 6.0")
            break
        elseif choice == "9" then
            geometryFormulas()
        elseif choice == "10" then
            algebraIdentities()
        else
            print("⚠️ Lựa chọn không hợp lệ. Thử lại.")
        end
    end
end

-- Bắt đầu chương trình
menu()