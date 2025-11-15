# easy_crack

**Платформа**: reversing.kr  
**Категория**: Rev  
**Сложность**: ★☆☆

## Условие
Windows crackme (`Easy_CrackMe.exe`).  
Запускается диалог, вводишь пароль — проверяет на правильность.  
Флаг/пароль — строка длиной ~12 символов.
![Reverse: функция проверки пароля](./assetss/incorrect.png)

**Скачать**: [reversing.kr → Challenge #1 (Easy Crack)](http://reversing.kr/challenge.php?ckattempt=1)

## Файлы
- `Easy_CrackMe.exe` — оригинальный бинарник
---

## Решение
Проверяем какой тип бинарника
```bash
$ file Easy_CrackMe.exe
PE32 executable (GUI) Intel 80386, for MS Windows
```
Закидываем в IDA наш бинарник и видим основную логику проверки пароля в функции
```c
Дизасм из IDA
int __cdecl sub_401080(HWND hDlg)
{
  CHAR String[97]; // [esp+4h] [ebp-64h] BYREF
  __int16 v3; // [esp+65h] [ebp-3h]
  char v4; // [esp+67h] [ebp-1h]

  memset(String, 0, sizeof(String));
  v3 = 0;
  v4 = 0;
  GetDlgItemTextA(hDlg, 1000, String, 100);
  if ( String[1] != 97 || strncmp(&String[2], Str2, 2u) || strcmp(&String[4], aR3versing) || String[0] != 69 ) "<--- тут у нас условие проверки пароля"
    return MessageBoxA(hDlg, aIncorrectPassw, Caption, 0x10u);
  MessageBoxA(hDlg, Text, Caption, 0x40u);
  return EndDialog(hDlg, 0);
}
```
функция sub_401080, принимающая дескриптор диалогового окна `HWND hDlg`, очищает стек `__cdecl`
. Она извлекает текст из поля ввода, проверяет его на соответствие жёстко закодированным условиям и выводит результат.

Давайте более кратко. у нас есть локальная переменная String которая проверяется условием `if ( String[1] != 97 || strncmp(&String[2], Str2, 2u) || strcmp(&String[4], aR3versing) || String[0] != 69 )` нам нужно сделать все наоборот, тк если условие выполнится, мы упадем в Incorrect

четко говорит, что `String[0] = 69 и String[1] = 97`
```python
chr(69) # 'E'
chr(97) # 'a'
```
Далее `strncmp(&String[2], Str2, 2u)`, которая просто сравнивает 2 символа начиная со 2 индекса со строкой `Str2 = '5y'`
И наконец `strcmp(&String[4], aR3versing)`, которая просто так же сравнивает начиная с 4 индекса со строкой `aR3versing = 'R3versing'`
Собираем флаг воедино, и получаем ответ :D

**Правильный пароль: `Ea5yR3versing`**
![Reverse: функция проверки пароля](./assetss/correct.png)
```text
Я так расписывал, тк многие функции из стандартной библиотеки kernel32 winapi для меня в новинку, надеюсь я подкачаю скилы в реверсе