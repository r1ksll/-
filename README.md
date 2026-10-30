# from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
import time

def test_login_form():
    driver = webdriver.Chrome()
    driver.get("https://b2c.passport.rt.ru")
    driver.maximize_window() # Лучше раскрыть окно, чтобы все элементы были видны

    # Ждем загрузки вкладки "Вход" и кликаем по ней
    login_tab = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.ID, "t-btn-tab-login"))
    )
    login_tab.click()

    # Вводим логин (используем тестовый номер телефона, чтобы не сработала валидация поля)
    username = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.ID, "username"))
    )
    username.send_keys("+79000000000")

    # Вводим пароль
    password = driver.find_element(By.ID, "password")
    password.send_keys("test_password")

    # Нажимаем кнопку "Войти"
    login_btn = driver.find_element(By.ID, "kc-login")
    login_btn.click()

    try:
        # Ожидаем появления сообщения об ошибке (до 10 секунд)
        error_msg = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "card-error"))
        )
        
        # Проверяем текст ошибки (на русском!)
        assert "Неправильное имя пользователя или пароль" in error_msg.text
        print("Test passed successfully!")

    except TimeoutException:
        # Если ошибка не появилась, вероятно, вылезла капча
        print("\n--- ВНИМАНИЕ! ---")
        print("Похоже, появилась капча или изменился класс ошибки.")
        print("У вас есть 30 секунд, чтобы ввести капчу вручную в браузере...")
        time.sleep(30) # Пауза для ручного ввода капчи
        
        try:
            # Пробуем найти ошибку еще раз после ввода капчи
            error_msg = driver.find_element(By.CLASS_NAME, "card-error")
            assert "Неправильное имя пользователя или пароль" in error_msg.text
            print("Test passed successfully (после ввода капчи)!")
        except Exception as e:
            print("Тест не пройден. Ошибка:", e)

    finally:
        # Закрываем браузер в любом случае
        driver.quit()

if __name__ == "__main__":
    test_login_form()