/**
 * Process Sandbox (process_sandbox.ts)
 *
 * Launch child processes with limited resources (time) and capture stdout/stderr.
 * This is a naive sandbox demo — for real isolation use containers, VMs, or OS-level seccomp.
 *
 * Usage:
 *  ts-node src/process_sandbox.ts "node -e \"console.log('hello')\""
 */
import { spawn } from 'child_process';

function runCmd(cmdline: string, timeoutMs = 2000) {
  const parts = cmdline.split(' ');
  const proc = spawn(parts[0], parts.slice(1), { stdio: ['ignore', 'pipe', 'pipe'] });
  const out: Buffer[] = [];
  const err: Buffer[] = [];
  proc.stdout.on('data', (b) => out.push(b));
  proc.stderr.on('data', (b) => err.push(b));
  const killTimer = setTimeout(() => {
    proc.kill('SIGKILL');
  }, timeoutMs);
  proc.on('close', (code, signal) => {
    clearTimeout(killTimer);
    console.log('exit', code, signal, 'stdout:', Buffer.concat(out).toString(), 'stderr:', Buffer.concat(err).toString());
  });
}

// CLI
const cmd = process.argv.slice(2).join(' ');
if (!cmd) { console.log('Usage: ts-node src/process_sandbox.ts <command>'); process.exit(1); }
runCmd(cmd, 3000);
