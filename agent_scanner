import os
import asyncio
import requests
from spade.agent import Agent
from spade.behaviour import OneShotBehaviour
from spade.message import Message

#Telegram:
TOKEN = "telegramToken"
chat_id = "chatId"
#filePath = "test.txt"

def sendTelegram(TOKEN, chat_id, filePath):
    url = f"https://api.telegram.org/bot{TOKEN}/sendDocument"
    files = {"document": open(filePath, "rb")}
    data = {"chat_id": chat_id}

    response = requests.post(url, data=data, files=files)
    print(f"{GREEN}Results successfully sent to Telegram:")
    print(response.json())
    
# ANSI escape codes for colored output
GREEN = "\033[92m"
PURPLE = "\033[95m"
RESET = "\033[0m"

# Crawler + Sorter Agent
class CrawlerAgent(Agent):
    class CrawlBehaviour(OneShotBehaviour):
        async def run(self):
            target_url = self.agent.target_url
            print(f"{GREEN}[+] Scanning {target_url} with Katana...{RESET}")
            os.system(f"katana -u {target_url} -silent -o raw_urls.txt")

            print(f"{GREEN}[+] Filtering URLs with parameters...{RESET}")
            filtered_urls = []
            with open("raw_urls.txt", "r") as file:
                for line in file:
                    url = line.strip()
                    if "?" in url or "=" in url:
                        filtered_urls.append(url)

            with open("filtered_urls.txt", "w") as file:
                for url in filtered_urls:
                    file.write(f"{url}\n")

            print(f"{PURPLE}[+] Found {len(filtered_urls)} URLs with parameters.{RESET}")
            print(f"{GREEN}[+] Running Gxss on filtered URLs...{RESET}")
            os.system(f"type filtered_urls.txt | Gxss -c 100 > gxss_urls.txt")

            with open("gxss_urls.txt", "r") as file:
                gxss_urls = [line.strip() for line in file.readlines()]
            
            print(f"{PURPLE}[+] Gxss found {len(gxss_urls)} attack points:{RESET}")
            for url in gxss_urls:
                print(url)
            
            with open("urls.txt", "w") as file:
                for url in gxss_urls:
                    file.write(f"{url}\n")
            
            print(f"{GREEN}[+] Crawling completed.{RESET}")
            
            # Sending Messege to XSSScannerAgent
            msg = Message(to=str(self.agent.scanner_jid))  # JID of scanner
            msg.set_metadata("performative", "inform")  
            msg.body = "Crawling completed, start scanning."
            await self.send(msg)
            
            self.kill()

    async def setup(self):
        print(f"{GREEN}[+] CrawlerAgent started.{RESET}")
        self.add_behaviour(self.CrawlBehaviour())
        self.web.start(hostname="127.0.0.1", port=10000)

# XSS scanner agent
class XSSScannerAgent(Agent):
    class ScanBehaviour(OneShotBehaviour):
        async def run(self):
            print(f"{GREEN}[+] Running DalFox for XSS scanning...{RESET}")
            os.system("dalfox file urls.txt > results.txt")

            sendTelegram(TOKEN, chat_id, "results.txt")

            with open("results.txt", "r") as file:
                results = file.read()

            if results.strip():
                print(f"{GREEN}[!] Vulnerabilities found:{RESET}\n{results}")
            else:
                print(f"{GREEN}[+] No vulnerabilities detected.{RESET}")
            
            self.kill()

    async def setup(self):
        print(f"{GREEN}[+] XSSScannerAgent started.{RESET}")
        self.add_behaviour(self.ScanBehaviour())
        self.web.start(hostname="127.0.0.1", port=10001)


async def main():
    target_url = input(f"{GREEN}Enter the target URL: {RESET}")
    
    crawler_jid = "crawler_agent@localhost"
    crawler_password = "*****"
    scanner_jid = "scanner_agent@localhost"
    scanner_password = "*****"
    
    crawler = CrawlerAgent(crawler_jid, crawler_password)
    crawler.target_url = target_url
    crawler.scanner_jid = scanner_jid
    
    scanner = XSSScannerAgent(scanner_jid, scanner_password)
    
    await crawler.start()
    await scanner.start()
    
    await asyncio.sleep(30)  # Time to finish tasks
    await crawler.stop()
    await scanner.stop()

if __name__ == "__main__":
    asyncio.run(main())
