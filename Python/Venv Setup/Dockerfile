# Multi-stage build for smaller image size
FROM python:3.13-alpine AS builder

WORKDIR /build

COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt gunicorn

FROM python:3.13-alpine AS runtime

# Uncomment the next line if you need ffmpeg
# RUN apk add --no-cache ffmpeg

COPY --from=builder /root/.local /root/.local

WORKDIR /backend

COPY . .

ENV PATH=/root/.local/bin:$PATH

EXPOSE 5001

CMD ["gunicorn", "--bind", "0.0.0.0:5001", "app:app"]