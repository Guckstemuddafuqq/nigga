#region

using System;
using System.Drawing;
using System.IO;
using System.Reflection;
using System.Timers;
using LeagueSharp;

#endregion

namespace ToasterLoading
{
    internal class Program
    {
        private const int WM_KEYUP = 0x101;
        private const int Disable = 0x20;

        private const int SecondsToWait = 250;

        private static MemoryStream _packet;
        private static bool _escaped;
        private static Timer _timer;

        private static void Main(string[] args)
        {
            _packet = new MemoryStream();
            Game.OnGameSendPacket += Game_OnGameSendPacket;
            Game.OnWndProc += Game_OnWndProc;
            Drawing.OnDraw += Drawing_OnDraw;
        }

        private static void Game_OnWndProc(WndEventArgs args)
        {
            try
            {
                if (args.Msg != WM_KEYUP || args.WParam != Disable)
                    return;

                _escaped = true;
                Game.SendPacket(_packet.ToArray(), PacketChannel.C2S, PacketProtocolFlags.Reliable);
                _packet.Close();
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.ToString());
            }
        }

        private static void Drawing_OnDraw(EventArgs args)
        {
            try
            {
                if (_escaped)
                    return;

                Drawing.DrawText(10, 10, Color.Green, Assembly.GetExecutingAssembly().GetName().Name);
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.ToString());
            }
        }

        private static void Game_OnGameSendPacket(GamePacketEventArgs args)
        {
            try
            {
                if (args.PacketData[0] != 189 || _escaped)
                    return;

                args.Process = false;
                _packet.Write(args.PacketData, 0, args.PacketData.Length);
                _timer = new Timer(SecondsToWait*1000);
                _timer.Elapsed += OnTimedEvent;
                _timer.Start();
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.ToString());
            }
        }

        private static void OnTimedEvent(object source, ElapsedEventArgs e)
        {
            try
            {
                if (!_escaped)
                {
                    _escaped = true;
                    Game.SendPacket(_packet.ToArray(), PacketChannel.C2S, PacketProtocolFlags.Reliable);
                    _packet.Close();
                }
                _timer.Close();
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.ToString());
            }
        }
    }
}
