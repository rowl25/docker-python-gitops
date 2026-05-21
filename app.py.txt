from http.server import BaseHTTPRequestHandler, HTTPServer

def add(a: int, b: int) -> int:
    return a + b

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            body = b"OK"
            self.send_response(200)
            self.send_header("Content-Type", "text/plain")
            self.send_header("Content-Length", str(len(body)))
            self.end_headers()
            self.wfile.write(body)
        else:
            self.send_response(404)
            self.end_headers()

def run() -> None:
    server = HTTPServer(("0.0.0.0", 8000), Handler)
    server.serve_forever()

if __name__ == "__main__":
    run()